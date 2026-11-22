---
name: backend-email
description: Use when implementing transactional email, email templates, drip sequences, onboarding emails, password reset, email verification, notification emails, or integrating with email providers (Resend, SendGrid, Postmark, Nodemailer). Also covers email deliverability, SPF/DKIM/DMARC setup, unsubscribe handling, and bounce management. Triggers on: email, transactional email, Resend, SendGrid, Postmark, Nodemailer, SMTP, email template, welcome email, password reset email, email verification, drip sequence, onboarding email, email deliverability, SPF, DKIM, DMARC, bounce, or any request to send automated email from an application.
---

# Backend Email Skill

You are a backend engineer and email deliverability specialist. You build email systems that reliably reach inboxes, never land in spam, and handle failures gracefully. You know that email is not "fire and forget" — it requires deliverability configuration, bounce handling, unsubscribe compliance, and template design that renders correctly across 15+ email clients. You build email infrastructure that scales from day 1 without surprises.

---

## Phase 1 — Provider Selection

| Provider | Best For | Pricing Model | Key Strength |
|---|---|---|---|
| **Resend** | Developer-first, React Email templates | Per email sent | React Email integration, excellent DX |
| **SendGrid** | High volume, marketing + transactional | Tiered by volume | Mature platform, analytics |
| **Postmark** | Transactional-only, high deliverability | Per email sent | Best deliverability for transactional |
| **Nodemailer + SES** | AWS-native, cost-sensitive | SES: $0.10/1000 | Cheapest at scale |
| **Mailgun** | Developer API, EU data residency | Per email sent | EU region, good API |

**Recommendation for new SaaS**: Resend for transactional (React Email templates), plus a marketing platform (Loops, Customer.io, or Brevo) for behavioral sequences.

---

## Phase 2 — Resend + React Email (Recommended Stack)

### Setup
```typescript
// lib/email/client.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY!);
```

### React Email Template
```tsx
// emails/WelcomeEmail.tsx
import {
  Body, Button, Container, Head, Heading, Html,
  Img, Link, Preview, Section, Text
} from '@react-email/components';

interface WelcomeEmailProps {
  firstName: string;
  loginUrl: string;
  unsubscribeUrl: string;
}

export function WelcomeEmail({ firstName, loginUrl, unsubscribeUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Welcome to YourApp — here's how to get started</Preview>
      <Body style={{ fontFamily: '-apple-system, sans-serif', backgroundColor: '#f9fafb' }}>
        <Container style={{ maxWidth: '580px', margin: '0 auto', padding: '40px 20px' }}>
          <Heading style={{ fontSize: '24px', color: '#111827' }}>
            Welcome, {firstName}!
          </Heading>
          <Text style={{ fontSize: '16px', color: '#374151', lineHeight: '1.6' }}>
            Your account is ready. Here's what to do first:
          </Text>
          <Section>
            <Text>✅ Complete your profile</Text>
            <Text>✅ Connect your first integration</Text>
            <Text>✅ Invite a teammate</Text>
          </Section>
          <Button
            href={loginUrl}
            style={{
              backgroundColor: '#3b82f6',
              color: '#ffffff',
              padding: '12px 24px',
              borderRadius: '6px',
              fontSize: '16px',
              fontWeight: '600',
              textDecoration: 'none',
              display: 'inline-block',
            }}
          >
            Go to your dashboard →
          </Button>
          <Text style={{ fontSize: '12px', color: '#9ca3af', marginTop: '40px' }}>
            YourApp · 123 Main St, San Francisco CA · 
            <Link href={unsubscribeUrl} style={{ color: '#9ca3af' }}>Unsubscribe</Link>
          </Text>
        </Container>
      </Body>
    </Html>
  );
}
```

### Sending Email
```typescript
// lib/email/send.ts
import { resend } from './client';
import { WelcomeEmail } from '@/emails/WelcomeEmail';
import { render } from '@react-email/render';

export async function sendWelcomeEmail(user: { email: string; firstName: string; id: string }) {
  const html = render(WelcomeEmail({
    firstName: user.firstName,
    loginUrl: `${process.env.APP_URL}/dashboard`,
    unsubscribeUrl: `${process.env.APP_URL}/unsubscribe?token=${generateUnsubscribeToken(user.id)}`,
  }));

  const { data, error } = await resend.emails.send({
    from: 'YourApp <hello@yourapp.com>',  // Always use your verified domain
    to: user.email,
    subject: 'Welcome to YourApp — your account is ready',
    html,
    tags: [
      { name: 'category', value: 'welcome' },
      { name: 'userId', value: user.id },
    ],
  });

  if (error) {
    console.error('Failed to send welcome email:', { userId: user.id, error });
    throw new Error(`Email send failed: ${error.message}`);
  }

  // Record email sent in DB for audit and unsubscribe tracking
  await db.emailLog.create({
    data: { userId: user.id, type: 'welcome', provider: 'resend', messageId: data.id }
  });
}
```

---

## Phase 3 — Essential Transactional Email Types

### Email Catalog (implement these for every SaaS)

| Email Type | Trigger | Key Content |
|---|---|---|
| Welcome | Sign up complete | First steps, login link |
| Email verification | Sign up | Verification link (expires 24h) |
| Password reset | Forgot password | Reset link (expires 1h), "Didn't request this?" note |
| Subscription activated | Payment success | Plan details, next billing date |
| Payment failed | Stripe `invoice.payment_failed` | Amount, update payment method link |
| Subscription cancelled | Downgrade/cancel | Effective date, reactivation offer |
| Trial ending | 3 days before trial expires | What they'll lose, upgrade CTA |
| Account security alert | New device login, password change | Timestamp, IP, "Wasn't you?" link |

### Password Reset Email
```typescript
// lib/email/templates/PasswordReset.tsx
export async function sendPasswordResetEmail(email: string, token: string) {
  const resetUrl = `${process.env.APP_URL}/reset-password?token=${token}`;

  await resend.emails.send({
    from: 'YourApp Security <security@yourapp.com>',
    to: email,
    subject: 'Reset your YourApp password',
    html: render(PasswordResetEmail({ resetUrl, expiresInHours: 1 })),
  });
}

// Always expire reset tokens after 1 hour
// Always invalidate the token after first use
// Always log password reset requests for security audit
```

---

## Phase 4 — Onboarding Email Sequence (Drip)

### 7-Day Onboarding Sequence
```
Day 0 (immediate): Welcome + first action CTA
Day 1: "Did you complete X?" — nudge for the #1 activation action
Day 3: Feature spotlight — the most valuable feature most users miss
Day 5: Social proof — customer success story relevant to their use case
Day 7: Check-in — "How's it going? Reply to this email" (builds replies → inbox placement)
Day 14: Trial ending reminder (if on trial)
```

### Scheduling with BullMQ
```typescript
import { emailQueue } from '@/lib/queues';
import { addDays } from 'date-fns';

export async function scheduleOnboardingSequence(userId: string, email: string) {
  const sequence = [
    { name: 'onboarding-day-1', delay: 24 * 60 * 60 * 1000 },   // 1 day
    { name: 'onboarding-day-3', delay: 3 * 24 * 60 * 60 * 1000 },
    { name: 'onboarding-day-5', delay: 5 * 24 * 60 * 60 * 1000 },
    { name: 'onboarding-day-7', delay: 7 * 24 * 60 * 60 * 1000 },
  ];

  for (const { name, delay } of sequence) {
    await emailQueue.add(name, { userId, email }, {
      delay,
      attempts: 3,
      backoff: { type: 'exponential', delay: 60_000 },
      jobId: `${name}-${userId}`,  // Idempotent — re-scheduling won't duplicate
    });
  }
}

// Cancel if user activates before sequence ends
export async function cancelOnboardingSequence(userId: string) {
  const jobNames = ['onboarding-day-1', 'onboarding-day-3', 'onboarding-day-5', 'onboarding-day-7'];
  for (const name of jobNames) {
    const job = await emailQueue.getJob(`${name}-${userId}`);
    if (job) await job.remove();
  }
}
```

---

## Phase 5 — Deliverability Configuration

### DNS Records (required for every sending domain)
```
# SPF — authorize your sending servers
TXT  yourdomain.com  "v=spf1 include:_spf.resend.com ~all"

# DKIM — sign emails cryptographically (get key from your provider dashboard)
TXT  resend._domainkey.yourdomain.com  "v=DKIM1; k=rsa; p=YOUR_PUBLIC_KEY"

# DMARC — policy for failed SPF/DKIM
TXT  _dmarc.yourdomain.com  "v=DMARC1; p=quarantine; rua=mailto:dmarc@yourdomain.com; pct=100"
```

### Deliverability Checklist
- [ ] SPF, DKIM, DMARC all configured and passing (verify at mail-tester.com)
- [ ] Sending from a subdomain for transactional (mail.yourapp.com) — protects the root domain reputation
- [ ] From address uses your verified domain — never gmail/yahoo sender addresses
- [ ] Unsubscribe link in every non-transactional email (CAN-SPAM / GDPR requirement)
- [ ] List-Unsubscribe header set for one-click unsubscribe
- [ ] Bounce and complaint webhooks configured — remove bounced addresses from sending list immediately
- [ ] Warm up new domain: start with low volume (100/day), increase 2× weekly

### Bounce & Complaint Handling
```typescript
// POST /api/webhooks/email-events (Resend webhook)
export async function handleEmailEvent(req: Request, res: Response) {
  const event = req.body;  // Verify signature first (Resend provides webhook signing)

  switch (event.type) {
    case 'email.bounced':
      // Hard bounce: permanently remove from list
      await db.emailSuppression.upsert({
        where: { email: event.data.to },
        create: { email: event.data.to, reason: 'bounce', suppressedAt: new Date() },
        update: { reason: 'bounce' },
      });
      break;

    case 'email.complained':
      // Spam complaint: immediately unsubscribe
      await db.emailSuppression.upsert({
        where: { email: event.data.to },
        create: { email: event.data.to, reason: 'complaint', suppressedAt: new Date() },
        update: { reason: 'complaint' },
      });
      break;
  }

  res.status(200).json({ received: true });
}

// Always check suppression list before sending
export async function isSuppressed(email: string): Promise<boolean> {
  const suppression = await db.emailSuppression.findUnique({ where: { email } });
  return !!suppression;
}
```

---

## Phase 6 — Email Template Best Practices

### Design Rules for Emails
- Max width: 600px — wider than this breaks in Outlook
- Use inline styles — CSS classes are stripped by most email clients
- Use table-based layout for complex structures — Flexbox/Grid are poorly supported
- Never use JavaScript — it's stripped by all email clients
- Images: always include `alt` text; always set `width` and `height` attributes
- Background images: not supported in Outlook — always provide a background color fallback
- Test in Litmus or Email on Acid before sending to a list
- Plain-text version: always include a plain-text alternative (Resend does this automatically with React Email)

---

## Non-Negotiable Standards

- Never send email without verifying your sending domain — your emails will land in spam.
- Always check the suppression list before sending — sending to bounced or complained addresses destroys deliverability.
- Password reset links expire in 1 hour and are single-use — invalidate on use.
- Every email that is not a direct response to a user action must include an unsubscribe link.
- Never log email body content — it may contain PII.
- Use idempotent job IDs for drip sequences — re-triggering sign-up logic should not send duplicate emails.
- Handle bounce and complaint webhooks from day 1 — not "later when it's a problem."
