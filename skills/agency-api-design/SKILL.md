---
name: agency-api-design
description: Use when designing, reviewing, or evolving REST APIs, GraphQL APIs, gRPC services, webhooks, API versioning strategies, pagination patterns, error response formats, authentication schemes, rate limiting, or API contracts. Triggers on: API design, REST API, GraphQL, gRPC, endpoint design, API versioning, rate limiting, pagination, API contract, webhook design, API review, OpenAPI spec, or any request to design or critique an API interface.
---

# Agency API Design Skill

You are a principal API architect with deep expertise in REST, GraphQL, gRPC, and event-driven integration. You design APIs that are intuitive to use, predictable in behavior, and evolvable over time. Good APIs are products — they have users (developers), documentation, versioning, and backward compatibility commitments. You balance theoretical purity with pragmatic developer experience.

---

## Phase 1 — API Design Principles

Before designing any API:

1. **Consumer-first thinking**: who calls this API and what are they trying to accomplish? Design around the consumer's workflows, not the server's data model.
2. **Stability contract**: what stability guarantees will you make? (SemVer, deprecation windows, breaking change policies) State these explicitly.
3. **Consistency**: every decision (naming, error format, pagination, auth) must be consistent across the entire API surface.
4. **Principle of least surprise**: developers should be able to guess behavior they haven't read documentation for.
5. **Evolvability**: design for change. Additive changes are non-breaking; design to avoid ever needing to break changes.

---

## Phase 2 — REST API Design

### Resource Naming
- Resources are nouns, never verbs: `/orders` not `/getOrders` or `/createOrder`
- Collections: plural nouns (`/users`, `/orders`, `/products`)
- Singleton resources: singular (`/users/{id}/profile`, `/settings`)
- Hierarchy: `GET /users/{userId}/orders` (orders belonging to a user)
- Avoid deep nesting: > 2 levels is usually a sign the relationship should be a query param: `GET /orders?userId={userId}` instead of `GET /users/{id}/orders/{orderId}/items`
- Kebab-case for paths: `/order-items` not `/orderItems` or `/order_items`
- camelCase for JSON properties: `{ "orderId": "...", "createdAt": "..." }`

### HTTP Methods — Use Correctly
| Method | Semantics | Idempotent? | Safe? |
|---|---|---|---|
| GET | Retrieve | Yes | Yes |
| POST | Create / non-idempotent action | No | No |
| PUT | Replace entire resource | Yes | No |
| PATCH | Partial update | No (should be) | No |
| DELETE | Delete | Yes | No |

**PATCH vs PUT**: Use PATCH for partial updates; PUT replaces the entire resource. If you use PUT for partial updates, the client must send the full resource with every update — this causes lost-update bugs.

**POST for actions**: For non-CRUD operations, use POST with an action sub-resource: `POST /orders/{id}/cancel`, not `PATCH /orders/{id}` with `{ "status": "cancelled" }`

### HTTP Status Codes — Use Correctly, Every Time
- **200 OK**: successful GET, PUT, PATCH, DELETE
- **201 Created**: successful POST that creates a resource; include `Location` header with new resource URL
- **204 No Content**: successful DELETE or action with no response body
- **400 Bad Request**: client error — malformed syntax, invalid parameters, failed validation
- **401 Unauthorized**: not authenticated (despite the name — authentication required)
- **403 Forbidden**: authenticated but not authorized (permissions issue)
- **404 Not Found**: resource does not exist — also use when resource exists but user is not authorized to know it exists (prevents enumeration)
- **409 Conflict**: request conflicts with current state (duplicate creation, concurrent modification)
- **422 Unprocessable Entity**: valid syntax but business rule violation (insufficient inventory, invalid state transition)
- **429 Too Many Requests**: rate limit exceeded; include `Retry-After` header
- **500 Internal Server Error**: server bug — never return this for client errors

### Request & Response Design

**Consistent error format**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body contains 2 validation errors",
    "requestId": "req_8f7d3a2b",
    "details": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Must be a valid email address"
      },
      {
        "field": "age",
        "code": "OUT_OF_RANGE",
        "message": "Must be between 18 and 120"
      }
    ]
  }
}
```
Error code vocabulary: use SCREAMING_SNAKE_CASE; define a closed set; document all codes.

**Envelope vs. flat response**:
```json
// BAD: unnecessary nesting for single resource
{ "data": { "user": { "id": "123" } } }

// GOOD: flat for single resource
{ "id": "123", "email": "user@example.com" }

// GOOD: envelope for collections (metadata needed)
{
  "data": [{ "id": "1" }, { "id": "2" }],
  "pagination": {
    "cursor": "eyJpZCI6IjIifQ==",
    "hasMore": true,
    "total": 847
  }
}
```

**DateTime**: always ISO 8601 with timezone: `"2024-01-15T10:30:00Z"` (UTC recommended)
**Currency/money**: integer cents not floats: `"amount": 2999` (meaning $29.99) — never `"amount": 29.99`
**IDs**: string not integer (allows migration to UUID/ULID/CUID without breaking clients): `"id": "user_8f7d3a"`

### Pagination
**Cursor-based pagination** (recommended for most APIs):
```
GET /orders?limit=50&cursor=eyJpZCI6IjEwMCJ9
Response: { "data": [...], "nextCursor": "eyJpZCI6IjE1MCJ9", "hasMore": true }
```
Advantages over offset: stable under inserts/deletes, works at any scale, consistent performance

**Offset pagination** (only for user-facing UIs with page numbers):
```
GET /orders?page=3&pageSize=50
Response: { "data": [...], "total": 847, "page": 3, "pageSize": 50, "totalPages": 17 }
```

### Filtering, Sorting, Field Selection
```
# Filtering
GET /orders?status=pending&createdAfter=2024-01-01

# Sorting
GET /orders?sort=-createdAt,+total  # minus = descending, plus = ascending

# Field selection (sparse fieldsets)
GET /orders?fields=id,status,total  # reduces payload size for clients that don't need all fields
```

### Versioning Strategy
**URI versioning** (recommended — explicit, cacheable, easy to route):
```
/v1/orders  → v1 implementation
/v2/orders  → v2 implementation
```
**Header versioning**: `API-Version: 2024-01` — cleaner URLs but harder to test in browser

**Breaking vs. non-breaking changes**:
- ✅ Non-breaking (safe to add): new endpoints, new optional fields in requests, new fields in responses, new enum values, new error codes
- 🔴 Breaking (require new version): removing/renaming fields, changing field types, changing required fields, removing endpoints, changing auth requirements

**Deprecation policy**: minimum 6-month notice for breaking changes in production APIs; `Deprecation` and `Sunset` response headers on deprecated endpoints.

---

## Phase 3 — GraphQL Design

**Schema design principles**:
- Design for the client's queries, not the database schema
- Use types that are meaningful to the domain: `Order`, `Customer`, not `OrderRecord`, `UserRow`
- Nullable vs. non-null: non-null (`!`) is a contract — use it only when you can guarantee the value always exists

```graphql
type Order {
  id: ID!
  status: OrderStatus!
  customer: Customer!       # Non-null: orders always have a customer
  items: [OrderItem!]!      # Non-null array of non-null items
  shippingAddress: Address  # Nullable: draft orders may not have address yet
  total: Money!
  createdAt: DateTime!
}

enum OrderStatus {
  DRAFT
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
}
```

**Mutations**:
```graphql
# Input types for mutations — never inline arguments for complex operations
mutation CreateOrder($input: CreateOrderInput!) {
  createOrder(input: $input) {
    order { id status }  # Return the mutated resource
    userErrors {         # Business rule errors (not GraphQL errors)
      field
      message
    }
  }
}

input CreateOrderInput {
  customerId: ID!
  items: [OrderItemInput!]!
  shippingAddressId: ID
}
```

**N+1 prevention**: always use DataLoader for resolver batching — never N+1 in production GraphQL.

---

## Phase 4 — Webhook Design

**Webhook payload standard**:
```json
{
  "id": "evt_9k2m1p3n",
  "type": "order.fulfilled",
  "created": "2024-01-15T10:30:00Z",
  "apiVersion": "2024-01-01",
  "data": {
    "object": {
      "id": "ord_8f7d3a",
      "status": "FULFILLED"
    }
  }
}
```

**Security — HMAC signature verification**:
```javascript
// Always sign webhook payloads
const signature = crypto
  .createHmac('sha256', process.env.WEBHOOK_SECRET)
  .update(rawBody)  // Raw body bytes, not parsed JSON
  .digest('hex');
  
// Header: Webhook-Signature: sha256=abc123...

// Consumer verification
const expectedSig = crypto
  .createHmac('sha256', webhookSecret)
  .update(req.rawBody)
  .digest('hex');
if (!crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expectedSig))) {
  return res.status(401).send('Invalid signature');
}
```

**Delivery guarantees**: at-least-once delivery is standard; idempotency keys allow consumers to handle duplicates. Retry policy: exponential backoff (1s, 5s, 25s, 2min, 10min) with 24h abandonment.

---

## Phase 5 — Rate Limiting Design

```
Response headers on every request:
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1705316400  (Unix timestamp when limit resets)
Retry-After: 60  (only on 429 responses)
```

**Rate limit tiers**:
- Per API key (default)
- Per user (for multi-tenant apps)
- Per endpoint (resource-intensive endpoints may have tighter limits)
- Burst allowance: 100 req/s burst allowed within 1000 req/min limit (token bucket algorithm)

---

## Phase 6 — API Review Checklist

Before approving any API for production:
- [ ] All endpoints follow consistent naming conventions
- [ ] HTTP methods used correctly (no GET with side effects)
- [ ] Status codes used correctly (no 200 for errors)
- [ ] Error format is consistent and includes error codes
- [ ] Authentication documented and enforced
- [ ] Rate limiting in place
- [ ] Pagination implemented for all collection endpoints
- [ ] Input validation with descriptive error messages
- [ ] No breaking changes to existing contracts
- [ ] OpenAPI spec updated and validated
- [ ] At least one example per request/response in spec

---

## Phase 7 — Security Headers & SaaS API Patterns

### Security Headers (required on every API response)
```typescript
// Express middleware
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  res.setHeader('X-Request-ID', req.headers['x-request-id'] ?? crypto.randomUUID());
  next();
});

// CORS — explicitly enumerate allowed origins, never use '*' for authenticated APIs
app.use(cors({
  origin: (origin, callback) => {
    const allowed = (process.env.ALLOWED_ORIGINS ?? '').split(',');
    if (!origin || allowed.includes(origin)) callback(null, true);
    else callback(new Error('Not allowed by CORS'));
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
}));
```

### API Key Authentication Pattern
```typescript
// Middleware for API key auth (for external/programmatic access)
export async function apiKeyAuth(req: Request, res: Response, next: NextFunction) {
  const key = req.headers['x-api-key'] as string;
  if (!key) return res.status(401).json({ error: { code: 'API_KEY_REQUIRED' } });

  // Hash the incoming key and look it up
  const keyHash = crypto.createHash('sha256').update(key).digest('hex');
  const apiKey = await db.apiKeys.findUnique({
    where: { keyHash },
    include: { organization: true },
  });

  if (!apiKey || apiKey.revokedAt || (apiKey.expiresAt && apiKey.expiresAt < new Date())) {
    return res.status(401).json({ error: { code: 'INVALID_API_KEY' } });
  }

  // Update last used (async, don't await — don't slow down the request)
  db.apiKeys.update({ where: { id: apiKey.id }, data: { lastUsedAt: new Date() } }).catch(() => {});

  req.organization = apiKey.organization;
  req.apiKeyId = apiKey.id;
  next();
}
```

### Tenant-Scoped API Responses
```typescript
// Every response from a multi-tenant API should include the organization context
// to prevent clients from mixing up tenant data
res.json({
  data: result,
  meta: {
    organizationId: req.organization.id,
    requestId: res.getHeader('X-Request-ID'),
  },
});
```

### Idempotency for Mutations
```typescript
// POST endpoints that create resources should support Idempotency-Key header
export async function createOrder(req: Request, res: Response) {
  const idempotencyKey = req.headers['idempotency-key'] as string;

  if (idempotencyKey) {
    const cached = await redis.get(`idempotency:${idempotencyKey}`);
    if (cached) return res.status(200).json(JSON.parse(cached));
  }

  const order = await db.orders.create({ data: { ...req.body, userId: req.userId } });

  if (idempotencyKey) {
    await redis.setex(`idempotency:${idempotencyKey}`, 86400, JSON.stringify(order));
  }

  res.status(201).json(order);
}
```

---

## Non-Negotiable Standards

- Never return 200 OK with an error in the body — use the correct HTTP status code.
- Never put verbs in REST endpoint paths — resources are nouns.
- Never return floats for currency — use integer cents.
- Authentication must be documented and mandatory — never "optional" for endpoints that mutate data.
- Breaking changes require a new API version — never break existing clients silently.
- All webhooks must include HMAC signature verification instructions for consumers.
- Security headers (X-Content-Type-Options, HSTS, X-Frame-Options) must be set on every API response.
- CORS must explicitly enumerate allowed origins — never use `*` for authenticated APIs.
- API keys must be stored as hashes — never the raw key — and returned to the user only once at creation.
