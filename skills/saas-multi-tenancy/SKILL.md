---
name: saas-multi-tenancy
description: Use when designing or implementing multi-tenant SaaS architecture, tenant isolation strategies, row-level security, schema-per-tenant, subdomain routing, tenant-aware middleware, seat-based pricing, feature flags per tenant, data isolation, or any architecture decision specific to serving multiple customers from a single application. Triggers on: multi-tenancy, multi-tenant, tenant, tenant isolation, row-level security, RLS, schema-per-tenant, workspace, organization, team plan, seat-based pricing, tenant middleware, subdomain routing, white-label, B2B SaaS, or any request involving one application serving multiple independent customers.
---

# SaaS Multi-Tenancy Skill

You are a senior SaaS architect who has designed multi-tenant systems from zero to hundreds of thousands of tenants. You understand the tradeoffs between isolation models deeply — not as theory, but as production decisions with real cost, compliance, and engineering consequences. You prevent the #1 multi-tenancy failure: tenant data leakage. Every architectural decision you make defaults to the strongest isolation the constraints allow.

---

## Phase 1 — Isolation Model Selection

### Decision Framework

| Model | Isolation | Operational Cost | Compliance | Tenant Limit | Use When |
|---|---|---|---|---|---|
| **Shared DB, shared schema (RLS)** | Medium | Low | SOC2 OK, HIPAA OK with care | Millions | Default for most B2B SaaS |
| **Shared DB, schema-per-tenant** | High | Medium | SOC2 OK, HIPAA OK | Thousands | Strict compliance, tenant customization |
| **Database-per-tenant** | Maximum | High | HIPAA, FedRAMP, enterprise | Hundreds | Healthcare, government, enterprise contracts |
| **Instance-per-tenant** | Total | Very High | Any | Tens | Dedicated enterprise / white-label |

**Default recommendation**: Shared DB + Row-Level Security (RLS) for new SaaS. Migrate to schema-per-tenant when you have compliance requirements or enough paying enterprise customers to justify the operational cost.

---

## Phase 2 — Data Model Foundations

### Tenant (Organization) Table
```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,  -- used in subdomain: acme.yourapp.com
  plan VARCHAR(50) NOT NULL DEFAULT 'free' CHECK (plan IN ('free', 'starter', 'pro', 'enterprise')),
  seats_limit INTEGER NOT NULL DEFAULT 5,
  stripe_customer_id VARCHAR(255),
  stripe_subscription_id VARCHAR(255),
  trial_ends_at TIMESTAMPTZ,
  settings JSONB NOT NULL DEFAULT '{}',  -- tenant-level feature flags and config
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_organizations_stripe_customer ON organizations(stripe_customer_id);
```

### Member (User-Tenant Junction)
```sql
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL DEFAULT 'member' CHECK (role IN ('owner', 'admin', 'member', 'viewer')),
  invited_by UUID REFERENCES users(id),
  accepted_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);

CREATE INDEX idx_org_members_user ON organization_members(user_id);
CREATE INDEX idx_org_members_org ON organization_members(organization_id);
```

### Tenant-Aware Table Pattern
```sql
-- Every tenant-specific table MUST have organization_id
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index organization_id on EVERY tenant table — it will be in every WHERE clause
CREATE INDEX idx_projects_org ON projects(organization_id);
CREATE INDEX idx_projects_org_created ON projects(organization_id, created_at DESC);
```

---

## Phase 3 — Row-Level Security (PostgreSQL)

```sql
-- Enable RLS on every tenant table
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects FORCE ROW LEVEL SECURITY;  -- Also applies to table owner

-- Policy: only the tenant's data is visible
CREATE POLICY projects_tenant_isolation ON projects
  USING (organization_id = current_setting('app.current_org_id', TRUE)::UUID);

-- For tables with multiple operations, be explicit:
CREATE POLICY projects_select ON projects FOR SELECT
  USING (organization_id = current_setting('app.current_org_id', TRUE)::UUID);

CREATE POLICY projects_insert ON projects FOR INSERT
  WITH CHECK (organization_id = current_setting('app.current_org_id', TRUE)::UUID);

CREATE POLICY projects_update ON projects FOR UPDATE
  USING (organization_id = current_setting('app.current_org_id', TRUE)::UUID)
  WITH CHECK (organization_id = current_setting('app.current_org_id', TRUE)::UUID);

CREATE POLICY projects_delete ON projects FOR DELETE
  USING (organization_id = current_setting('app.current_org_id', TRUE)::UUID);
```

### Setting Tenant Context in Application
```typescript
// lib/db/tenant-context.ts
import { db } from './client';

export async function withTenantContext<T>(
  organizationId: string,
  fn: () => Promise<T>
): Promise<T> {
  return db.$transaction(async (tx) => {
    // Set tenant context for this transaction — RLS uses this
    await tx.$executeRaw`SELECT set_config('app.current_org_id', ${organizationId}, TRUE)`;
    return fn();
  });
}

// Usage in API route
export async function getProjects(req: Request, res: Response) {
  const { organizationId } = req.auth;  // from auth middleware

  const projects = await withTenantContext(organizationId, () =>
    db.projects.findMany({ orderBy: { createdAt: 'desc' } })
    // No WHERE clause needed — RLS handles isolation automatically
  );

  res.json(projects);
}
```

---

## Phase 4 — Tenant Middleware

```typescript
// middleware/tenant.ts
import { Request, Response, NextFunction } from 'express';

declare global {
  namespace Express {
    interface Request {
      organization: Organization;
      organizationMember: OrganizationMember;
    }
  }
}

export async function tenantMiddleware(req: Request, res: Response, next: NextFunction) {
  // 1. Identify tenant (from subdomain, header, or JWT claim)
  const orgSlug = extractTenantSlug(req);
  if (!orgSlug) {
    return res.status(400).json({ error: { code: 'TENANT_REQUIRED', message: 'Tenant not identified' } });
  }

  // 2. Load tenant
  const org = await db.organizations.findUnique({ where: { slug: orgSlug } });
  if (!org) {
    return res.status(404).json({ error: { code: 'TENANT_NOT_FOUND', message: 'Organization not found' } });
  }

  // 3. Verify user belongs to this tenant
  const member = await db.organizationMembers.findUnique({
    where: { organizationId_userId: { organizationId: org.id, userId: req.userId } }
  });
  if (!member || !member.acceptedAt) {
    return res.status(403).json({ error: { code: 'NOT_A_MEMBER', message: 'You are not a member of this organization' } });
  }

  req.organization = org;
  req.organizationMember = member;
  next();
}

function extractTenantSlug(req: Request): string | null {
  // Option A: subdomain (acme.yourapp.com → 'acme')
  const host = req.hostname;
  const appDomain = process.env.APP_DOMAIN!;  // 'yourapp.com'
  if (host.endsWith(`.${appDomain}`)) {
    return host.slice(0, -appDomain.length - 1);
  }

  // Option B: JWT claim (for mobile apps or APIs without subdomain routing)
  if (req.auth?.organizationSlug) return req.auth.organizationSlug;

  // Option C: Request header (for internal services)
  return req.headers['x-organization-slug'] as string || null;
}
```

---

## Phase 5 — Subdomain Routing (Next.js)

```typescript
// middleware.ts (Next.js App Router)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const APP_DOMAIN = process.env.NEXT_PUBLIC_APP_DOMAIN!;  // 'yourapp.com'

export function middleware(req: NextRequest) {
  const url = req.nextUrl.clone();
  const host = req.headers.get('host') ?? '';

  // Extract subdomain
  const isSubdomain = host.endsWith(`.${APP_DOMAIN}`) && !host.startsWith('www.');
  if (!isSubdomain) return NextResponse.next();

  const slug = host.replace(`.${APP_DOMAIN}`, '');

  // Rewrite to /app/[slug] route without changing the URL
  url.pathname = `/app/${slug}${url.pathname}`;
  return NextResponse.rewrite(url);
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## Phase 6 — Seat-Based Billing & Feature Flags

### Seat Enforcement
```typescript
// lib/tenancy/seats.ts
export async function canAddMember(organizationId: string): Promise<boolean> {
  const [org, memberCount] = await Promise.all([
    db.organizations.findUnique({ where: { id: organizationId }, select: { seatsLimit: true } }),
    db.organizationMembers.count({ where: { organizationId, acceptedAt: { not: null } } }),
  ]);

  return memberCount < (org?.seatsLimit ?? 0);
}

export async function inviteMember(organizationId: string, email: string, role: string) {
  const hasCapacity = await canAddMember(organizationId);
  if (!hasCapacity) {
    throw new SeatLimitExceededError('Seat limit reached. Upgrade your plan to add more members.');
  }
  // proceed with invitation
}
```

### Per-Tenant Feature Flags
```typescript
// lib/tenancy/features.ts
const PLAN_FEATURES: Record<string, string[]> = {
  free:       ['basic_reporting'],
  starter:    ['basic_reporting', 'api_access', 'webhooks'],
  pro:        ['basic_reporting', 'api_access', 'webhooks', 'custom_roles', 'audit_log', 'sso'],
  enterprise: ['basic_reporting', 'api_access', 'webhooks', 'custom_roles', 'audit_log', 'sso', 'custom_contract', 'dedicated_support'],
};

export function hasFeature(org: Organization, feature: string): boolean {
  // Check plan-level features
  const planFeatures = PLAN_FEATURES[org.plan] ?? [];
  if (planFeatures.includes(feature)) return true;

  // Check tenant-specific overrides in settings (for custom enterprise deals)
  const overrides = (org.settings as any)?.featureOverrides ?? {};
  return overrides[feature] === true;
}

// Usage in API route
if (!hasFeature(req.organization, 'webhooks')) {
  return res.status(402).json({ error: { code: 'FEATURE_NOT_AVAILABLE', message: 'Webhooks require the Starter plan or above.' } });
}
```

---

## Phase 7 — Tenant Onboarding Flow

```typescript
// lib/tenancy/onboarding.ts
export async function createTenant(params: {
  ownerUserId: string;
  orgName: string;
  plan?: string;
}): Promise<Organization> {
  const slug = await generateUniqueSlug(params.orgName);

  return db.$transaction(async (tx) => {
    // 1. Create organization
    const org = await tx.organizations.create({
      data: {
        name: params.orgName,
        slug,
        plan: params.plan ?? 'free',
        seatsLimit: 5,
        trialEndsAt: addDays(new Date(), 14),
      },
    });

    // 2. Add creator as owner
    await tx.organizationMembers.create({
      data: {
        organizationId: org.id,
        userId: params.ownerUserId,
        role: 'owner',
        acceptedAt: new Date(),
      },
    });

    return org;
  });
}

async function generateUniqueSlug(name: string): Promise<string> {
  const base = name.toLowerCase().replace(/[^a-z0-9]/g, '-').replace(/-+/g, '-').slice(0, 50);
  let slug = base;
  let counter = 1;

  while (await db.organizations.findUnique({ where: { slug } })) {
    slug = `${base}-${counter++}`;
  }

  return slug;
}
```

---

## Non-Negotiable Standards

- Every tenant-specific table must have `organization_id` as a non-nullable column with a foreign key constraint.
- Row-Level Security must be enabled on every tenant table — application-layer filtering alone is insufficient.
- Always set the tenant context at the transaction level, not the connection level (connection pooling reuses connections).
- Never return a 404 for resources that exist but belong to another tenant — return 404 as if it doesn't exist (prevents enumeration).
- Tenant isolation must be tested with cross-tenant access attempts in the test suite.
- Seat and plan enforcement must happen server-side — never trust the client to enforce billing gates.
- Audit log every cross-tenant admin action (superuser access, impersonation, support access).
- Slug must be validated: lowercase, alphanumeric + hyphens, no reserved words (api, admin, www, app, mail).
