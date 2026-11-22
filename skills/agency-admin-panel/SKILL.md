---
name: agency-admin-panel
description: Use when designing, specifying, reviewing, or building an admin panel or back-office dashboard for a software product. Covers the 6 essential admin modules: user management, content management (CMS), payments and transactions, notifications, reports and analytics, and roles and permissions. Also covers admin panel UX patterns, security hardening, audit logging, and technology choices. Triggers on: admin panel, admin dashboard, back-office, user management, CMS admin, payment dashboard, admin roles, permissions system, admin analytics, or any request to build or design an administrative interface for a product.
---

# Agency Admin Panel Skill

You are a senior product engineer and system architect specializing in internal tools and administrative interfaces. You build admin panels that give operations teams genuine power — not watered-down interfaces that require a developer for every change. A great admin panel reduces developer interrupt-driven work by 80%, enables non-technical operators to run the business, and maintains a complete audit trail of every action. A poor admin panel creates security risks, operational bottlenecks, and support nightmares.

---

## Phase 1 — Admin Panel Architecture Principles

Before designing any module:

1. **Operator-first design**: the admin panel is a product for your internal team. Apply the same UX rigor you'd apply to your customer-facing product — your operators deserve good UX too.
2. **Principle of least privilege**: every operator should have exactly the access they need, nothing more. Design roles before designing screens.
3. **Auditability by default**: every write operation (create, update, delete) must generate an audit log entry. You will need this for compliance, debugging, and incident investigation.
4. **Reversibility where possible**: delete operations should be soft-deletes; changes should be revertible; bulk actions need confirmation modals with specific consequences stated.
5. **Performance at scale**: admin panels must handle large data volumes. Build pagination, search, and filtering from day one — not as an afterthought.

### Technology Stack Recommendations

**For new products (fastest to production)**:
- Retool, Appsmith, Tooljet (no-code admin builders) — fastest to production, limited customization
- AdminJS (Node.js) — auto-generates admin from your data models
- Django Admin (Python) — battle-tested, highly customizable

**For products needing full custom control**:
- React + TanStack Table + shadcn/ui (frontend) + your existing API (backend)
- Next.js App Router with server components for admin routes
- Separate deployment from your main app (security: different auth, different domain)

**Admin domain recommendation**: always serve admin at a separate subdomain or path with server-side authentication, not just client-side checks.
- `admin.yourdomain.com` or `yourdomain.com/admin`
- Block admin routes at the infrastructure level from public internet if possible (IP allowlist or VPN requirement)

---

## Module 1 — User Management

**Core requirement**: "You should be able to see all your users, verify, block, or manage them."

### User List View
Required columns (configurable):
| Column | Type | Sortable | Filterable |
|---|---|---|---|
| ID | text | Yes | Yes (exact) |
| Name | text | Yes | Yes (contains) |
| Email | text | Yes | Yes (contains, exact) |
| Status | enum | Yes | Yes (multi-select) |
| Created At | datetime | Yes | Yes (date range) |
| Last Login | datetime | Yes | Yes (date range) |
| Plan/Role | enum | Yes | Yes (multi-select) |
| Actions | — | No | No |

Pagination: cursor-based, 25/50/100 per page options, total count displayed.

### User Detail View
Every user record must expose:
- **Identity**: name, email, phone, profile data
- **Status**: active/suspended/banned/pending verification + status history
- **Activity**: login history (IP, device, timestamp), key actions
- **Subscriptions/billing**: current plan, payment history, next billing date
- **Linked data**: orders, projects, tickets — whatever entities belong to this user
- **Notes**: internal admin notes (timestamped, with admin name)

### User Actions
Every admin action must have: confirmation dialog, reason field (for auditing), and audit log entry.

| Action | Confirmation Required | Reversible | Audit Required |
|---|---|---|---|
| Verify email | Yes | No | Yes |
| Suspend account | Yes + reason | Yes (unsuspend) | Yes |
| Permanently ban | Yes + reason | No (can be unreviewed) | Yes |
| Reset password | Yes | N/A | Yes |
| Impersonate user | Yes + reason | — | Yes (critical) |
| Delete account | Yes + type DELETE | Soft delete only | Yes |

**Impersonation feature** (critical for support):
Allow admins to log in as a user to reproduce their exact experience. Requirements:
- Explicit consent language in confirmation ("You are about to view the app as [user]")
- Visual indicator in the UI when in impersonation mode (banner: "Viewing as [user] — click to exit")
- Impersonation session logged with start time, end time, admin ID, and pages visited
- Impersonation cannot be used to perform irreversible financial actions

### User Export
- CSV export of filtered user set (all visible columns)
- Include privacy filters: exclude PII fields by default; require explicit inclusion
- Export must be logged (who exported, what filter, when)

---

## Module 2 — Content Management (CMS)

**Core requirement**: "Change banners, categories, products, and listings without updating the app."

### What belongs in the CMS
Any content that changes more than once per year belongs in the CMS, not in code:
- Marketing banners and announcements
- Featured product/listing configurations
- Category taxonomy
- Homepage hero content
- Promotional pricing and discount codes
- Email template content (subject lines, body text — not layout)
- Help center / FAQ content
- Legal pages (Terms, Privacy Policy — these change regularly)

### CMS Design Principles

**Live preview**: every content change must be previewable before publishing. Operators should never publish blind.

**Publishing workflow**:
```
Draft → Review → Scheduled / Published
```
- Draft: saved but not live
- Review: sent for approval (optional, for high-stakes content)
- Scheduled: will go live at a specific datetime
- Published: live immediately

**Content versioning**: maintain previous 10 versions of every content item. Enable one-click rollback to any previous version.

**Rich text handling**: use a battle-tested rich text editor (TipTap, Quill, Lexical). Never allow raw HTML input from admin users — this is an XSS vector. Sanitize all output.

### CMS Entity Types

**Banner/Announcement**:
```
Fields: Title, Body (rich text), CTA text, CTA URL, Image, 
        Target audience (all/segment), Active from, Active until, Priority order
```

**Product/Listing**:
```
Fields: Name, Description, Images (drag-to-reorder), Category, Tags,
        Status (active/draft/archived), Price, Featured (boolean), Sort order
Bulk actions: bulk archive, bulk category reassign, bulk export
```

**Category Taxonomy**:
```
Fields: Name, Slug (auto-generated from name), Parent category, Image, 
        Description, Display order, SEO title/description
```

---

## Module 3 — Payments & Transactions

**Core requirement**: "Track every payment, refund, and failed transaction, and your commission easily."

### Transaction List View
Columns: Transaction ID, User, Amount, Currency, Status, Payment Method, Date, Gateway Reference

**Status types** (use consistent terminology):
- `succeeded` / `pending` / `failed` / `refunded` / `partially_refunded` / `disputed`

**Filters** (must have all):
- Date range (presets: today, 7 days, 30 days, custom)
- Status (multi-select)
- Payment method (card, PayPal, bank transfer, etc.)
- Amount range (min/max)
- User email/ID search

### Transaction Detail View
For each transaction:
- Full transaction details (amount, fees, net, gateway, timestamp)
- User info with link to user record
- Payment method details (last 4 digits, expiry — never full card numbers)
- Timeline: attempt → authorization → capture → settlement (or failure at any stage)
- Gateway transaction ID and link to Stripe/PayPal dashboard
- Related transactions (refunds, disputes linked to original)
- Raw webhook payload (collapsed by default — for debugging)

### Refund Workflow
```
Admin clicks "Issue Refund" →
  Modal: 
    - Refund amount (pre-filled with max refundable, editable)
    - Partial or full refund selection
    - Reason (required dropdown: duplicate, fraudulent, requested_by_customer, other)
    - Internal notes (required)
  Confirmation: "Refund $X.XX to [user] via [gateway]. This cannot be undone."
  → Calls payment gateway API
  → Updates transaction status
  → Sends refund confirmation email to customer
  → Creates audit log entry
```

### Revenue Dashboard
Real-time metrics displayed prominently:
- Today's revenue, 7-day revenue, MRR (for subscription products)
- Failed payment rate (target < 2%)
- Refund rate (target < 1%)
- Dispute rate (target < 0.3%)
- Charts: revenue trend, failed vs. succeeded by day

**Commission tracking** (for marketplace products):
- Platform fee per transaction (configurable %)
- Payout due to vendors/creators
- Payout history and status

---

## Module 4 — Notifications

**Core requirement**: "Send offers, updates and announcements to your users, anytime."

### Notification Types

**Push notifications** (web + mobile):
- Compose: title (50 chars), body (200 chars), URL/deep link, image
- Targeting: all users, by plan, by tag, by last activity date, by geography
- Scheduling: send now or schedule for specific date/time
- Preview on device type before sending

**In-app notifications** (bell icon):
- Target users who are currently logged in
- Rich content: formatted text, action buttons

**Email campaigns** (operational, not marketing):
- Triggered vs. broadcast
- Use for: system announcements, maintenance notices, feature launches
- Note: bulk marketing emails belong in a dedicated email platform (Mailchimp, Klaviyo), not the admin panel

### Notification Compose UI
```
Step 1: Content
  - Type: Push / In-app / Email
  - Subject/Title
  - Body (with preview of rendered output)
  - CTA button (optional): text + URL

Step 2: Audience
  - All users: [X] users will receive this
  - Filter by: [plan / status / tag / last active / geography]
  - Estimated reach updates as filters are applied

Step 3: Schedule
  - Send now vs. Schedule for [datetime]
  - Timezone: use user's local timezone or one specific timezone

Step 4: Review & Send
  - Summary: "Sending to [X] users at [time]"
  - Final confirmation with large count displayed
  - Cannot be recalled once sent — make this clear
```

### Notification History
- Full log: sent notifications, delivery status, open rates, click rates
- Failed delivery count with reason
- Ability to resend to failed recipients only

---

## Module 5 — Reports & Analytics

**Core requirement**: "Download your business reports yourself."

### Standard Reports (available without developer involvement)

**Daily Operations Report**:
- New signups today vs. 7-day average
- Revenue today vs. yesterday vs. 7-day average
- Support tickets opened/closed
- Failed payments requiring follow-up

**User Growth Report** (exportable, date-range selectable):
- New users per day/week/month
- Activation rate (signed up → completed key action)
- Retention curve (Day 1, 7, 30, 90)
- Churn rate (subscriptions)

**Revenue Report** (exportable):
- Gross revenue, refunds, net revenue by period
- Revenue by plan tier
- MRR, ARR (subscription products)
- LTV by cohort

**Export Formats**: CSV (for Excel analysis), PDF (for executive sharing), JSON (for data engineers)

### Report Builder (self-serve)
For non-standard reports, provide a simple filter-and-export UI:
- Select entity: Users / Orders / Transactions / Products
- Add filters: same filter set as the list views
- Select columns to include
- Export as CSV

This eliminates 80% of "can you pull this report for me?" developer requests.

---

## Module 6 — Roles & Permissions

**Core requirement**: "Don't give every team member full access. Decide who can see and do what."

### Permission Model Design

**Role-based access control (RBAC) with custom roles**:

**Default roles** (customize for your product):
| Role | Description | Can Access |
|---|---|---|
| Super Admin | Full access + billing + team management | Everything |
| Admin | Full operational access | All modules except billing and team management |
| Operations | Day-to-day management | Users, content, notifications, basic reports |
| Support | Customer support tasks | User view, limited user actions (no ban, no delete), transaction view |
| Finance | Financial data | Transactions, refunds, revenue reports only |
| Content | Content management | CMS module only |
| Read Only | View access for stakeholders | All modules, no write access |

**Permission granularity** (for each module):
```
users.view        — can see user list
users.view_pii    — can see email, phone (separate from view)
users.edit        — can update user fields
users.suspend     — can suspend/unsuspend
users.ban         — can permanently ban
users.delete      — can delete accounts
users.impersonate — can log in as user (high privilege — rarely grant)
users.export      — can export user data

[Same pattern for: transactions, content, notifications, reports]
```

### Role Management UI
- Create custom roles with specific permission selections
- Assign admin users to roles
- View all admins with their role and last activity
- Role audit: see which permissions are assigned to which roles

### Admin Account Security
- Mandatory 2FA for all admin accounts (TOTP or hardware key)
- Session timeout after 1 hour of inactivity
- IP allowlist option (restrict admin access to office IPs or VPN)
- Failed login lockout after 5 attempts
- Password rotation policy: every 90 days

---

## Phase 2 — Audit Log (Cross-Cutting Concern)

Every admin action must be logged:
```
{
  "id": "audit_8f7d3a",
  "timestamp": "2024-01-15T10:30:00Z",
  "adminId": "admin_123",
  "adminEmail": "sarah@company.com",
  "action": "users.suspend",
  "resourceType": "user",
  "resourceId": "user_456",
  "reason": "Violation of terms of service — spam reports",
  "metadata": {
    "previousStatus": "active",
    "newStatus": "suspended"
  },
  "ipAddress": "203.0.113.1",
  "userAgent": "Mozilla/5.0..."
}
```

Audit logs must be:
- Immutable: no admin can delete or edit audit logs
- Retained: minimum 1 year (3 years for financial operations)
- Searchable: by admin, action type, resource, date range
- Exportable: for compliance and incident investigation

---

## Non-Negotiable Standards

- Never allow admin panel access without 2FA — no exceptions.
- All admin actions that mutate data require confirmation and are recorded in audit logs.
- Delete operations must be soft deletes with a data retention policy — permanent deletes require a secondary confirmation.
- Admin panel must be served from a different auth context than the main app — a compromised user session should not grant admin access.
- PII visibility must be a separate permission from general user view — not all support staff need to see email addresses.
- Payment refunds must require a reason field that is logged — this is both a compliance and abuse prevention requirement.
- Bulk actions (bulk suspend, bulk delete) must show the exact count and require typing a confirmation phrase, not just clicking OK.
