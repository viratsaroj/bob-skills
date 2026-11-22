---
name: backend-payments
description: Use when implementing payment processing, subscription billing, webhooks, cron jobs, background jobs, authentication (JWT, sessions, OAuth), or any backend integration with Stripe, Razorpay, or other payment providers. Covers the full backend layer: auth middleware, payment flows, webhook verification, idempotency, refunds, subscription lifecycle, scheduled tasks, and job queues. Triggers on: Stripe, Razorpay, payment, checkout, subscription, billing, webhook, cron job, background job, JWT, session, OAuth, authentication, auth middleware, sign in, sign up, refresh token, job queue, BullMQ, or any request to handle money, auth, or scheduled work on the backend.
---

# Backend & Payments Skill

You are a senior backend engineer who has implemented payments and auth in production SaaS products. You know that money and authentication are the two areas where a single bug causes irreversible damage — data loss, fraud, security breaches, or financial liability. You implement these with defense-in-depth: idempotency, HMAC verification, audit logs, and test coverage on every branch. You never cut corners on security for speed.

---

## Phase 1 — Authentication

### JWT Strategy
```typescript
// lib/auth/jwt.ts
import jwt from 'jsonwebtoken';

const ACCESS_TOKEN_SECRET = process.env.JWT_ACCESS_SECRET!;
const REFRESH_TOKEN_SECRET = process.env.JWT_REFRESH_SECRET!;

export function signAccessToken(userId: string): string {
  return jwt.sign({ sub: userId, type: 'access' }, ACCESS_TOKEN_SECRET, {
    expiresIn: '15m',  // Short-lived — if stolen, expires quickly
    issuer: 'your-app',
    audience: 'your-app-users',
  });
}

export function signRefreshToken(userId: string): string {
  return jwt.sign({ sub: userId, type: 'refresh' }, REFRESH_TOKEN_SECRET, {
    expiresIn: '30d',  // Long-lived — stored in httpOnly cookie
    issuer: 'your-app',
    audience: 'your-app-users',
  });
}

export function verifyAccessToken(token: string): { sub: string } {
  return jwt.verify(token, ACCESS_TOKEN_SECRET, {
    issuer: 'your-app',
    audience: 'your-app-users',
  }) as { sub: string };
}
```

### Auth Middleware
```typescript
// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import { verifyAccessToken } from '@/lib/auth/jwt';

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: { code: 'UNAUTHORIZED', message: 'Missing or invalid Authorization header' } });
  }

  const token = authHeader.slice(7);
  try {
    const payload = verifyAccessToken(token);
    req.userId = payload.sub;
    next();
  } catch (err) {
    return res.status(401).json({ error: { code: 'TOKEN_EXPIRED', message: 'Access token expired — please refresh' } });
  }
}

// Role-based access control
export function requireRole(role: 'admin' | 'member') {
  return async (req: Request, res: Response, next: NextFunction) => {
    const user = await getUserById(req.userId);
    if (user.role !== role && user.role !== 'admin') {
      return res.status(403).json({ error: { code: 'FORBIDDEN', message: 'Insufficient permissions' } });
    }
    next();
  };
}
```

### Password Hashing
```typescript
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12;  // 10–14 is the production range

export async function hashPassword(plaintext: string): Promise<string> {
  return bcrypt.hash(plaintext, SALT_ROUNDS);
}

export async function verifyPassword(plaintext: string, hash: string): Promise<boolean> {
  return bcrypt.compare(plaintext, hash);
}
// Never: MD5, SHA-1, SHA-256 alone for passwords — always bcrypt/argon2/scrypt
```

### HttpOnly Cookie — Storing Refresh Tokens Securely

The refresh token must **never** go to `localStorage` or the response body for client-side storage. Set it as an `httpOnly` cookie — JavaScript cannot read it, so XSS attacks cannot steal it.

```typescript
// lib/auth/cookies.ts
import { Response, Request } from 'express';

const IS_PROD = process.env.NODE_ENV === 'production';

export function setRefreshTokenCookie(res: Response, refreshToken: string): void {
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,          // JS cannot read this cookie — blocks XSS token theft
    secure: IS_PROD,         // HTTPS only in production
    sameSite: 'strict',      // Never sent on cross-site requests — blocks CSRF
    maxAge: 30 * 24 * 60 * 60 * 1000,  // 30 days in ms
    path: '/api/auth',       // Scoped to auth routes only — not sent on every request
  });
}

export function clearRefreshTokenCookie(res: Response): void {
  res.clearCookie('refreshToken', {
    httpOnly: true,
    secure: IS_PROD,
    sameSite: 'strict',
    path: '/api/auth',
  });
}

export function getRefreshTokenFromCookie(req: Request): string | undefined {
  return req.cookies?.refreshToken;
}
```

### Login — Issue Both Tokens
```typescript
// POST /api/auth/login
export async function login(req: Request, res: Response) {
  const { email, password } = req.body;

  const user = await getUserByEmail(email);
  if (!user || !(await verifyPassword(password, user.passwordHash))) {
    return res.status(401).json({ error: { code: 'INVALID_CREDENTIALS', message: 'Email or password is incorrect' } });
  }

  const accessToken = signAccessToken(user.id);
  const refreshToken = signRefreshToken(user.id);

  // Store refresh token hash in DB (for rotation/revocation)
  const tokenHash = crypto.createHash('sha256').update(refreshToken).digest('hex');
  await db.refreshTokens.create({
    data: { userId: user.id, tokenHash, expiresAt: addDays(new Date(), 30) }
  });

  // Refresh token → httpOnly cookie (never in response body)
  setRefreshTokenCookie(res, refreshToken);

  // Access token → response body (client stores in memory only)
  res.json({
    accessToken,         // Client holds this in memory / React state — never localStorage
    user: { id: user.id, email: user.email, name: user.name },
  });
}
```

### Token Refresh — Rotate the Cookie
```typescript
// POST /api/auth/refresh
export async function refreshTokens(req: Request, res: Response) {
  const incomingRefreshToken = getRefreshTokenFromCookie(req);

  if (!incomingRefreshToken) {
    return res.status(401).json({ error: { code: 'NO_REFRESH_TOKEN' } });
  }

  // Verify the JWT
  let payload: { sub: string };
  try {
    payload = jwt.verify(incomingRefreshToken, process.env.JWT_REFRESH_SECRET!) as { sub: string };
  } catch {
    clearRefreshTokenCookie(res);
    return res.status(401).json({ error: { code: 'INVALID_REFRESH_TOKEN' } });
  }

  // Check DB — is this token still valid? (not revoked, not already used)
  const tokenHash = crypto.createHash('sha256').update(incomingRefreshToken).digest('hex');
  const storedToken = await db.refreshTokens.findUnique({ where: { tokenHash } });

  if (!storedToken || storedToken.revokedAt || storedToken.expiresAt < new Date()) {
    // Reuse detected — invalidate entire family (potential token theft)
    await db.refreshTokens.updateMany({
      where: { userId: payload.sub },
      data: { revokedAt: new Date() },
    });
    clearRefreshTokenCookie(res);
    return res.status(401).json({ error: { code: 'REFRESH_TOKEN_REUSED' } });
  }

  // Rotate: revoke old token, issue new pair
  await db.refreshTokens.update({ where: { tokenHash }, data: { revokedAt: new Date() } });

  const newAccessToken = signAccessToken(payload.sub);
  const newRefreshToken = signRefreshToken(payload.sub);

  const newTokenHash = crypto.createHash('sha256').update(newRefreshToken).digest('hex');
  await db.refreshTokens.create({
    data: { userId: payload.sub, tokenHash: newTokenHash, expiresAt: addDays(new Date(), 30) }
  });

  setRefreshTokenCookie(res, newRefreshToken);  // New cookie replaces old
  res.json({ accessToken: newAccessToken });
}
```

### Logout — Clear Cookie and Revoke Token
```typescript
// POST /api/auth/logout
export async function logout(req: Request, res: Response) {
  const refreshToken = getRefreshTokenFromCookie(req);

  if (refreshToken) {
    const tokenHash = crypto.createHash('sha256').update(refreshToken).digest('hex');
    await db.refreshTokens.updateMany({ where: { tokenHash }, data: { revokedAt: new Date() } });
  }

  clearRefreshTokenCookie(res);  // Remove the cookie from the browser
  res.status(204).send();
}
```

### Next.js App Router Pattern (Server Actions / Route Handlers)
```typescript
// app/api/auth/login/route.ts
import { cookies } from 'next/headers';
import { NextResponse } from 'next/server';

export async function POST(req: Request) {
  const { email, password } = await req.json();
  // ... validate credentials ...

  const refreshToken = signRefreshToken(user.id);
  const accessToken = signAccessToken(user.id);

  // Set httpOnly cookie via Next.js cookies() API
  cookies().set('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 30 * 24 * 60 * 60,  // seconds in Next.js (not ms)
    path: '/api/auth',
  });

  return NextResponse.json({ accessToken, user: { id: user.id, email: user.email } });
}
```

### Cookie vs. localStorage — Why httpOnly Wins
| | `httpOnly` Cookie | `localStorage` |
|---|---|---|
| XSS attack steals token | ❌ No — JS cannot read it | ✅ Yes — `localStorage` is fully readable by JS |
| CSRF attack sends token | Mitigated by `SameSite=Strict` | ❌ Not applicable (sent in header manually) |
| Persists across tabs | ✅ Yes | ✅ Yes |
| Survives page refresh | ✅ Yes | ✅ Yes |
| Works in SSR/Server Components | ✅ Yes — readable server-side | ❌ No — browser-only API |
| Sent automatically | ✅ Yes (to same origin) | ❌ No — must attach manually |

**Rule**: access token in memory (React state), refresh token in `httpOnly` cookie. Never store either in `localStorage`.

### Session Security Checklist
- [ ] Refresh tokens stored in `httpOnly`, `Secure`, `SameSite=Strict` cookies — never `localStorage`
- [ ] Access tokens in memory only (React state/closure) — never persisted to storage
- [ ] Refresh token hash stored in DB — enables revocation and rotation tracking
- [ ] Refresh token rotation: every `/api/auth/refresh` issues a new token and revokes the old one
- [ ] Refresh token reuse detection: if old token is reused, revoke all tokens for that user immediately
- [ ] Cookie scoped to `/api/auth` path — not sent on every API request
- [ ] Rate limiting on `/auth/login`: max 10 attempts per IP per 15 minutes
- [ ] Audit log every login, logout, password change, and token refresh

---

## Phase 2 — Stripe Integration

### Setup
```typescript
// lib/stripe.ts
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-06-20',  // Always pin the API version
  typescript: true,
});
```

### One-Time Payment Checkout
```typescript
// POST /api/payments/create-checkout-session
export async function createCheckoutSession(req: Request, res: Response) {
  const { priceId, userId } = req.body;

  const session = await stripe.checkout.sessions.create({
    mode: 'payment',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.APP_URL}/payment/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/payment/cancelled`,
    client_reference_id: userId,  // Link session back to your user
    metadata: { userId },          // Also store in metadata for webhook retrieval
  });

  res.json({ url: session.url });
}
```

### Subscription Billing
```typescript
// POST /api/subscriptions/create
export async function createSubscription(req: Request, res: Response) {
  const { userId, priceId } = req.body;

  // Get or create Stripe customer
  let user = await getUserById(userId);
  if (!user.stripeCustomerId) {
    const customer = await stripe.customers.create({
      email: user.email,
      metadata: { userId },
    });
    await updateUser(userId, { stripeCustomerId: customer.id });
    user = { ...user, stripeCustomerId: customer.id };
  }

  // Create checkout session for subscription
  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    customer: user.stripeCustomerId,
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.APP_URL}/dashboard?subscription=success`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14,  // optional
      metadata: { userId },
    },
  });

  res.json({ url: session.url });
}
```

### Webhook Handler (Critical — most payment logic lives here)
```typescript
// POST /api/webhooks/stripe
import { stripe } from '@/lib/stripe';

const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET!;

export async function stripeWebhook(req: Request, res: Response) {
  // 1. Verify signature FIRST — before any processing
  const sig = req.headers['stripe-signature'] as string;
  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(req.rawBody, sig, WEBHOOK_SECRET);
    // req.rawBody must be the raw Buffer — never the parsed JSON
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // 2. Acknowledge receipt immediately — Stripe retries if no 2xx within 30s
  res.status(200).json({ received: true });

  // 3. Process asynchronously (or use a job queue for heavy work)
  try {
    await processStripeEvent(event);
  } catch (err) {
    console.error('Webhook processing error:', { eventId: event.id, type: event.type, err });
    // Do NOT return 500 here — that would cause Stripe to retry, potentially double-processing
    // Instead: log to error monitoring, alert on-call
  }
}

async function processStripeEvent(event: Stripe.Event) {
  // Idempotency: check if this event was already processed
  const alreadyProcessed = await db.webhookEvents.findUnique({ where: { stripeEventId: event.id } });
  if (alreadyProcessed) return;  // Safe to skip — already handled

  // Record event (idempotency key)
  await db.webhookEvents.create({ data: { stripeEventId: event.id, type: event.type } });

  switch (event.type) {
    case 'checkout.session.completed': {
      const session = event.data.object as Stripe.Checkout.Session;
      const userId = session.metadata?.userId;
      if (session.mode === 'subscription') {
        await activateSubscription(userId!, session.subscription as string);
      } else {
        await fulfillOrder(userId!, session.id);
      }
      break;
    }
    case 'customer.subscription.updated': {
      const sub = event.data.object as Stripe.Subscription;
      await updateSubscriptionStatus(sub.metadata.userId, sub.status, sub.current_period_end);
      break;
    }
    case 'customer.subscription.deleted': {
      const sub = event.data.object as Stripe.Subscription;
      await cancelSubscription(sub.metadata.userId);
      break;
    }
    case 'invoice.payment_failed': {
      const invoice = event.data.object as Stripe.Invoice;
      await handlePaymentFailure(invoice.customer as string, invoice.subscription as string);
      break;
    }
    case 'invoice.payment_succeeded': {
      const invoice = event.data.object as Stripe.Invoice;
      await recordSuccessfulPayment(invoice);
      break;
    }
  }
}
```

---

## Phase 3 — Razorpay Integration

```typescript
// lib/razorpay.ts
import Razorpay from 'razorpay';
import crypto from 'crypto';

export const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID!,
  key_secret: process.env.RAZORPAY_KEY_SECRET!,
});

// Create order (server-side — never trust client amounts)
export async function createRazorpayOrder(amountInPaise: number, currency = 'INR', userId: string) {
  return razorpay.orders.create({
    amount: amountInPaise,  // Always in smallest unit (paise for INR)
    currency,
    receipt: `order_${userId}_${Date.now()}`,
    notes: { userId },
  });
}

// Verify payment signature (MUST verify before fulfilling order)
export function verifyRazorpaySignature(
  orderId: string,
  paymentId: string,
  signature: string
): boolean {
  const body = `${orderId}|${paymentId}`;
  const expectedSignature = crypto
    .createHmac('sha256', process.env.RAZORPAY_KEY_SECRET!)
    .update(body)
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// POST /api/payments/verify-razorpay
export async function verifyRazorpayPayment(req: Request, res: Response) {
  const { razorpay_order_id, razorpay_payment_id, razorpay_signature, userId } = req.body;

  const isValid = verifyRazorpaySignature(razorpay_order_id, razorpay_payment_id, razorpay_signature);
  if (!isValid) {
    return res.status(400).json({ error: { code: 'INVALID_SIGNATURE', message: 'Payment verification failed' } });
  }

  // Safe to fulfill — signature verified
  await fulfillOrder(userId, razorpay_order_id, razorpay_payment_id);
  res.json({ success: true });
}
```

---

## Phase 4 — Webhooks (Generic Pattern)

```typescript
// Generic HMAC webhook verification
export function verifyWebhookSignature(
  payload: Buffer,
  signature: string,
  secret: string,
  algorithm: 'sha256' | 'sha512' = 'sha256'
): boolean {
  const expected = crypto
    .createHmac(algorithm, secret)
    .update(payload)
    .digest('hex');

  // Always use timingSafeEqual to prevent timing attacks
  try {
    return crypto.timingSafeEqual(
      Buffer.from(signature.replace(/^sha256=/, '')),
      Buffer.from(expected)
    );
  } catch {
    return false;
  }
}

// Webhook processing checklist:
// 1. Verify signature FIRST
// 2. Return 200 immediately
// 3. Check idempotency (has this event ID been processed?)
// 4. Process in background job if heavy
// 5. Never retry on failure without understanding the failure
```

---

## Phase 5 — Cron Jobs & Background Jobs

### Node.js Cron (node-cron)
```typescript
import cron from 'node-cron';

// Run at 2am UTC every day
cron.schedule('0 2 * * *', async () => {
  console.log('[cron] Starting daily cleanup job');
  try {
    await cleanupExpiredSessions();
    await sendSubscriptionRenewalReminders();
    console.log('[cron] Daily cleanup completed');
  } catch (err) {
    console.error('[cron] Daily cleanup failed:', err);
    // Alert: send to Slack/PagerDuty
    await alertOnCall('daily-cleanup-failed', err);
  }
}, { timezone: 'UTC' });
```

### BullMQ (Redis-backed job queue — recommended for production)
```typescript
import { Queue, Worker } from 'bullmq';
import { redis } from '@/lib/redis';

// Define queues
export const emailQueue = new Queue('email', { connection: redis });
export const paymentQueue = new Queue('payment-webhooks', { connection: redis });

// Add a job
await emailQueue.add('send-welcome', { userId, email }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 5000 },
  removeOnComplete: 100,  // Keep last 100 completed jobs for debugging
  removeOnFail: 500,
});

// Process jobs
const emailWorker = new Worker('email', async (job) => {
  if (job.name === 'send-welcome') {
    await sendWelcomeEmail(job.data.userId, job.data.email);
  }
}, { connection: redis, concurrency: 5 });

emailWorker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err);
});
```

### Cron Job Checklist
- [ ] Every cron job has a timeout — never let a runaway job run forever
- [ ] Every cron job has error handling and alerting on failure
- [ ] Cron jobs are idempotent — safe to run twice if they overlap or retry
- [ ] Cron jobs do not run in every pod/instance in a multi-instance deployment (use distributed locking or a dedicated cron runner)
- [ ] Cron schedule is in UTC, not server local time

---

## Phase 6 — Idempotency

```typescript
// Every payment-related POST endpoint should support idempotency keys
export async function createPayment(req: Request, res: Response) {
  const idempotencyKey = req.headers['idempotency-key'] as string;

  if (idempotencyKey) {
    // Check if we've already processed this key
    const existing = await db.idempotencyKeys.findUnique({ where: { key: idempotencyKey } });
    if (existing) {
      return res.status(200).json(existing.responseBody);  // Return cached response
    }
  }

  // Process the payment
  const result = await processPayment(req.body);

  // Cache the response
  if (idempotencyKey) {
    await db.idempotencyKeys.create({
      data: { key: idempotencyKey, responseBody: result, expiresAt: addDays(new Date(), 1) }
    });
  }

  res.json(result);
}
```

---

## Non-Negotiable Standards

- Never trust client-side amounts — always look up the price from the database on the server.
- Always verify webhook signatures before processing — never skip this step.
- Always use idempotency keys on payment operations — network retries can cause double charges.
- Never log full credit card numbers, CVVs, or raw payment tokens — PCI compliance.
- Refresh tokens must be stored in httpOnly cookies, never localStorage.
- Use `crypto.timingSafeEqual` for all signature comparisons — prevents timing attacks.
- Every payment event must be recorded in an audit log with timestamp, userId, amount, and outcome.
- Cron jobs must be idempotent — if they run twice, the result must be the same as running once.
- Test every webhook event type with Stripe CLI or Razorpay test mode before going live.
