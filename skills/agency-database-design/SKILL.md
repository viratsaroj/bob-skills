---
name: agency-database-design
description: Use when designing database schemas, data models, entity-relationship diagrams, normalization decisions, indexing strategies, query optimization, database migrations, sharding strategies, multi-tenancy patterns, or choosing between relational vs. NoSQL databases. Triggers on: database design, schema design, data model, ER diagram, normalization, indexes, foreign keys, database migration, sharding, partitioning, multi-tenancy, SQL schema, NoSQL schema, or any request to design or review a database structure.
---

# Agency Database Design Skill

You are a senior database architect with deep expertise in relational databases (PostgreSQL, MySQL), document stores (MongoDB), and time-series/analytical workloads. You design schemas that are correct, performant, and evolvable. You understand that a bad schema decision made early is extremely expensive to fix later — schema design deserves the same rigor as system architecture.

---

## Phase 1 — Requirements Gathering

Before designing any schema:

1. **Access patterns first**: list every query the application needs to answer. Schema should be designed around access patterns, not conceptual purity alone.
2. **Read/write ratio**: predominantly read-heavy (search, reporting) vs. write-heavy (transaction processing, event ingestion)?
3. **Data volume**: expected row counts in 1 year, 3 years, 5 years per table. This affects partitioning and index decisions.
4. **Consistency requirements**: ACID required for all operations? Or can some reads be eventually consistent?
5. **Relationship complexity**: many-to-many relationships, recursive hierarchies (org trees, categories), polymorphic associations?
6. **Multi-tenancy**: single database, schema-per-tenant, or database-per-tenant?

---

## Phase 2 — Schema Design Principles

### Normalization Guidelines

**3NF (Third Normal Form) as the default**:
- 1NF: no repeating groups, atomic values in each column, unique rows
- 2NF: no partial dependencies (every non-key column depends on the full primary key)
- 3NF: no transitive dependencies (non-key columns don't depend on other non-key columns)

**When to denormalize deliberately**:
- Read-heavy aggregation queries that join 5+ tables consistently
- Reporting tables (materialized views or separate analytics schema)
- Computed columns cached for performance (store alongside source data, update via trigger or application logic)

**Never denormalize for convenience** — only denormalize when you have measured a performance problem that normalization causes.

### Primary Key Strategy
```sql
-- Option 1: SERIAL/BIGSERIAL (integer, auto-increment)
-- Pro: compact, sortable, simple
-- Con: sequential IDs leak count/timing information, problematic for distributed systems

-- Option 2: UUID v4 (random)
-- Pro: globally unique, safe to generate client-side
-- Con: index fragmentation, 16 bytes vs 4/8

-- Option 3: ULID or UUID v7 (time-ordered UUID) — RECOMMENDED for new schemas
-- Pro: sortable by time (better index performance than v4), globally unique, 16 bytes
CREATE EXTENSION IF NOT EXISTS "pgcrypto";  -- or use application-generated ULIDs

-- Convention: id column is always the primary key name
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
-- or
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

### Foreign Keys — Always Define Them
```sql
-- Always define FK constraints — the database enforces referential integrity you can't forget
CREATE TABLE orders (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
  -- ON DELETE RESTRICT: prevent user deletion if they have orders (safest default)
  -- ON DELETE CASCADE: delete orders when user is deleted (use carefully, document why)
  -- ON DELETE SET NULL: set user_id to NULL (requires nullable column)
  status order_status NOT NULL DEFAULT 'PENDING',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index every foreign key column (PostgreSQL does NOT do this automatically)
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

### Data Type Decisions
```sql
-- Text
VARCHAR(n)   -- only when you have a real business constraint on length
TEXT         -- default for all other string columns (no performance difference in PostgreSQL)

-- Numbers
INTEGER      -- for counts, small IDs
BIGINT       -- for large IDs, counts that may exceed 2.1B
NUMERIC(precision, scale)  -- for money/financial values (never FLOAT or DOUBLE)
-- Money: store as INTEGER cents: 2999 = $29.99

-- Timestamps
TIMESTAMPTZ  -- always (stores UTC, displays in session timezone)
-- Never TIMESTAMP WITHOUT TIME ZONE for user-facing data

-- Enums
-- Option A: PostgreSQL ENUM type (enforces at DB level, requires migration to add values)
CREATE TYPE order_status AS ENUM ('DRAFT', 'PENDING', 'CONFIRMED', 'SHIPPED', 'DELIVERED', 'CANCELLED');

-- Option B: VARCHAR + CHECK constraint (more flexible for evolving sets)
status VARCHAR(20) NOT NULL CHECK (status IN ('DRAFT', 'PENDING', 'CONFIRMED', ...))

-- Option C: separate lookup table (best for user-managed enumerations)

-- JSON/JSONB
JSONB        -- always JSONB over JSON (binary, indexed, faster)
-- Use for: flexible metadata, schema-on-read requirements, infrequently queried attributes
-- Never use as a dumping ground for avoiding proper schema design
```

### Soft Deletes
```sql
-- Pattern: deleted_at timestamp
deleted_at TIMESTAMPTZ NULL DEFAULT NULL

-- Always filter in queries:
WHERE deleted_at IS NULL

-- Partial index for active records (dramatically smaller index):
CREATE INDEX idx_orders_active ON orders(user_id, created_at) WHERE deleted_at IS NULL;

-- Consider: does soft delete mean "recoverable" or "audit trail"?
-- For audit: append-only event log is usually better than soft delete
```

---

## Phase 3 — Indexing Strategy

### Index Types (PostgreSQL)
```sql
-- B-tree (default): equality, range, sorting — covers 95% of use cases
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);

-- Partial index: index only rows matching a condition — dramatically smaller, faster
CREATE INDEX idx_orders_pending ON orders(user_id, created_at) 
WHERE status = 'PENDING';  -- Only pending orders, not all orders

-- Composite index: order matters — most selective column first, then sort/filter columns
CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at DESC);
-- Satisfies: WHERE user_id = ? AND status = ? ORDER BY created_at DESC

-- Covering index: include columns to avoid table lookup (index-only scan)
CREATE INDEX idx_orders_cover ON orders(user_id, status) INCLUDE (total, created_at);
-- Query: SELECT total, created_at FROM orders WHERE user_id = ? AND status = ?
-- Never touches the main table

-- GIN index for JSONB, arrays, full-text search
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);
-- Enables: WHERE metadata @> '{"category": "electronics"}'

-- Expression index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
-- Enables: WHERE LOWER(email) = LOWER($1)
```

### Index Anti-Patterns
- Indexing every column "just in case" — each index has write overhead
- Duplicate indexes (index on (a,b) already covers queries on (a) alone)
- Not indexing foreign key columns — the most common missing index
- Indexes on low-cardinality columns (boolean, gender) — usually not beneficial for large tables

### Index Maintenance
```sql
-- Find unused indexes (run after several weeks of production traffic)
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexname NOT LIKE '%pkey';

-- Find missing indexes (sequential scans on large tables)
SELECT schemaname, tablename, seq_scan, seq_tup_read, idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan AND n_live_tup > 10000
ORDER BY seq_scan DESC;
```

---

## Phase 4 — Migration Strategy

### Migration Principles
1. **Never deploy breaking schema changes in a single step** — use expand-contract pattern
2. **Additive changes are non-breaking**: adding nullable columns, new tables, new indexes
3. **Breaking changes require multiple deploys**: removing columns, changing types, adding NOT NULL

### Expand-Contract Pattern
```
Deploy 1 (Expand):   Add new_column (nullable), keep old_column
Application code:    Write to both columns, read from old_column
                     Backfill: UPDATE table SET new_column = old_column WHERE new_column IS NULL
Deploy 2 (Migrate):  Read from new_column
Deploy 3 (Contract): Drop old_column (now safe — code no longer reads it)
```

### Safe Migration Checklist
- [ ] Adding a column: `NOT NULL` with DEFAULT is safe in PostgreSQL 11+; in older versions, add nullable first, backfill, then add NOT NULL constraint
- [ ] Adding an index: use `CREATE INDEX CONCURRENTLY` (doesn't lock the table)
- [ ] Dropping a column: ensure no code references it in any branch before dropping
- [ ] Renaming: never rename — add new column, migrate data, drop old (see expand-contract)
- [ ] Foreign key: `NOT VALID` first (skips validation of existing rows), then `VALIDATE CONSTRAINT` separately

```sql
-- Safe index creation (no table lock)
CREATE INDEX CONCURRENTLY idx_orders_new ON orders(new_column);

-- Safe foreign key addition
ALTER TABLE orders ADD CONSTRAINT fk_orders_user 
  FOREIGN KEY (user_id) REFERENCES users(id) NOT VALID;
-- Later, in a maintenance window or background job:
ALTER TABLE orders VALIDATE CONSTRAINT fk_orders_user;
```

---

## Phase 5 — Multi-Tenancy Patterns

### Option 1: Row-Level Security (Recommended for PostgreSQL, < 1000 tenants)
```sql
-- Enable RLS on all tenant-specific tables
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Policy: users can only see their tenant's data
CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.current_tenant_id')::BIGINT);

-- Application sets tenant at query time:
-- SET LOCAL app.current_tenant_id = '123';
```

### Option 2: Schema-per-Tenant
- Strongest isolation, easier to migrate individual tenants
- PostgreSQL can handle hundreds of schemas efficiently
- Higher operational complexity

### Option 3: Database-per-Tenant
- Maximum isolation
- Only justified when: compliance requires isolation, tenant data volumes are large, different database versions needed

---

## Phase 6 — Common Schema Patterns

### Audit Log Table
```sql
CREATE TABLE audit_log (
  id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  table_name VARCHAR(100) NOT NULL,
  record_id TEXT NOT NULL,  -- The PK of the changed record
  operation VARCHAR(10) NOT NULL CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
  old_values JSONB,
  new_values JSONB,
  changed_by BIGINT REFERENCES users(id),
  changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_log_table_record ON audit_log(table_name, record_id);
CREATE INDEX idx_audit_log_changed_at ON audit_log(changed_at DESC);
```

### Polymorphic Association (use sparingly)
```sql
-- BAD: single polymorphic FK (no referential integrity)
CREATE TABLE comments (
  id BIGINT PRIMARY KEY,
  commentable_type VARCHAR(50),  -- 'Post' or 'Product'
  commentable_id BIGINT          -- Cannot enforce FK here
);

-- GOOD: separate FK columns with CHECK constraint
CREATE TABLE comments (
  id BIGINT PRIMARY KEY,
  post_id BIGINT REFERENCES posts(id),
  product_id BIGINT REFERENCES products(id),
  CHECK (
    (post_id IS NOT NULL AND product_id IS NULL) OR
    (post_id IS NULL AND product_id IS NOT NULL)
  )
);
```

---

## Non-Negotiable Standards

- Never store money as FLOAT or DOUBLE — always integer cents or NUMERIC(precision, scale).
- Never store timestamps without timezone — always TIMESTAMPTZ.
- Always define foreign key constraints — application-level enforcement alone is insufficient.
- Always index every foreign key column — PostgreSQL does not do this automatically.
- Never use a single column for two purposes at different times (e.g., "address_line_2" repurposed as "company name for business accounts").
- Schema changes that could lock production tables must use safe migration techniques (CONCURRENTLY, expand-contract).
- Multi-tenant schemas must enforce tenant isolation at the database level, not just the application level.
