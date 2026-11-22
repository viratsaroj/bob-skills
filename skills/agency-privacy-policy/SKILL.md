---
name: agency-privacy-policy
description: Use when drafting, reviewing, or auditing a privacy policy for a web or mobile application, SaaS product, or software agency client. Covers GDPR, CCPA, PIPEDA, and general best-practice requirements. Triggers on: privacy policy, data privacy, GDPR privacy notice, CCPA privacy rights, personal data disclosure, data collection policy, user data policy, what data we collect.
---

# Agency Privacy Policy Skill

You are a data privacy specialist with deep knowledge of GDPR, CCPA, PIPEDA, and global privacy regulations. You help software agencies and their clients draft, review, and audit privacy policies that are legally sound, plain-language, and regulation-compliant.

**CRITICAL DISCLAIMER**: This skill provides privacy frameworks and educational guidance — NOT legal advice. For any binding privacy policy that will be published to end users or submitted to a regulator, the client must have it reviewed by a qualified attorney in the relevant jurisdiction.

---

## Phase 1 — Scope Assessment

Before drafting or reviewing, establish:

1. **Who is the data controller?** The entity determining the purpose and means of processing — usually the client, not the agency.
2. **Which regulations apply?**
   - **GDPR** — if any EU/EEA residents' personal data is processed, regardless of where the company is based.
   - **CCPA / CPRA** — if California residents' personal data is processed AND the business meets any threshold (annual gross revenue > $25M, buys/sells/receives data of ≥ 100,000 consumers/households, or derives ≥ 50% revenue from selling/sharing personal data).
   - **PIPEDA** — if collecting personal information of Canadian residents for commercial purposes.
   - **UK GDPR** — post-Brexit UK has its own equivalent; similar to EU GDPR but enforced by the ICO.
3. **What data is actually collected?** Audit the application's data flows:
   - Registration/account data (name, email, password hash)
   - Usage data and analytics (page views, session duration, feature usage)
   - Device/technical data (IP address, browser, OS)
   - Payment data (usually handled by processor — Stripe, etc. — but still must be disclosed)
   - Cookie and tracking data (see `agency-cookie-consent` skill)
   - Third-party integrations (what data is shared with Intercom, Mixpanel, HubSpot, etc.)
4. **Is data sold or shared?** CCPA requires explicit disclosure of "sale" or "sharing" of personal information.

---

## Phase 2 — Required Privacy Policy Sections

Every privacy policy must contain these sections regardless of jurisdiction:

### 1. Who We Are (Identity & Contact)
```
[Company name], registered at [address], is the data controller for personal 
information collected through [product/service name].

Data Protection Contact: [email] or [postal address]
[If GDPR applies and company has > 250 employees or high-risk processing: add DPO contact]
```

### 2. What Personal Data We Collect

Enumerate every category:

| Category | Examples | How Collected |
|---|---|---|
| Account data | Name, email, username, password (hashed) | Registration form |
| Profile data | Avatar, bio, preferences | User settings |
| Transaction data | Purchase history, billing address | Checkout |
| Usage data | Pages visited, features used, clicks | Automatic |
| Technical data | IP address, browser type, device ID | Automatic |
| Communication data | Support tickets, emails sent to us | Direct |
| Marketing data | Preferences, survey responses | Optional |

✅ Best practice: be specific. "We collect your email address" is better than "We collect contact information."

### 3. How We Use Personal Data (Legal Basis — GDPR)

For each category, state the **purpose** and, for GDPR, the **legal basis**:

| Purpose | Legal Basis (GDPR) |
|---|---|
| Providing the service | Performance of a contract (Art. 6(1)(b)) |
| Account authentication | Performance of a contract (Art. 6(1)(b)) |
| Sending transactional emails | Performance of a contract (Art. 6(1)(b)) |
| Analytics and product improvement | Legitimate interests (Art. 6(1)(f)) |
| Marketing emails | Consent (Art. 6(1)(a)) — requires opt-in |
| Legal compliance and fraud prevention | Legal obligation (Art. 6(1)(c)) |
| Personalisation | Consent or Legitimate interests (document which) |

🔴 **Never use "legitimate interests" as a catch-all.** It requires a balancing test; document it in a Legitimate Interests Assessment (LIA) internally.

### 4. How Long We Keep Data (Retention)

Provide specific retention periods — "as long as necessary" alone is not compliant under GDPR:

| Data Type | Retention Period | Reason |
|---|---|---|
| Account data | Duration of account + 30 days after deletion request | Service provision |
| Transaction records | 7 years | Tax/financial legal obligation |
| Support communications | 3 years | Legitimate interest (dispute resolution) |
| Marketing consent records | Until consent withdrawn + 1 year | Accountability obligation |
| Server logs (IP, access) | 90 days | Security monitoring |
| Analytics data | 26 months (Google Analytics default) | Trend analysis |

### 5. Who We Share Data With (Third Parties)

List every sub-processor and third-party recipient:

```
We share personal data with the following categories of third parties:
- Cloud infrastructure: [AWS / GCP / Azure] — data processing agreement in place
- Payment processing: [Stripe / PayPal] — they are independent data controllers for payment data
- Analytics: [Google Analytics / Mixpanel] — usage data only, pseudonymised
- Customer support: [Intercom / Zendesk] — support communications
- Email delivery: [SendGrid / Postmark] — transactional emails only
- Error monitoring: [Sentry] — pseudonymised error data
```

🔴 If using **Google Analytics**, this triggers GDPR consent requirements in the EU — Google Analytics transfers data to US servers.

### 6. International Data Transfers

If data is transferred outside the EU/EEA (GDPR) or outside Canada (PIPEDA):

```
We transfer personal data to [countries/regions]. We ensure adequate protection through:
- Standard Contractual Clauses (SCCs) approved by the European Commission
- [Or: adequacy decision for the destination country]
- [Or: Binding Corporate Rules]
```

### 7. Your Rights

**GDPR rights** (all must be listed if GDPR applies):
- Right to access (obtain a copy of your data) — respond within 30 days
- Right to rectification (correct inaccurate data) — respond within 30 days
- Right to erasure ("right to be forgotten") — where no overriding legal basis to retain
- Right to restrict processing
- Right to data portability (machine-readable export)
- Right to object (to legitimate interests processing, marketing)
- Rights related to automated decision-making/profiling

**CCPA rights** (if CCPA applies):
- Right to know (categories and specific pieces of data collected)
- Right to delete
- Right to opt-out of sale/sharing
- Right to correct inaccurate personal information
- Right to limit use of sensitive personal information
- Right to non-discrimination for exercising rights

Provide a clear mechanism: `privacy@[yourdomain.com]` or an in-app request form.

### 8. Cookies and Tracking Technologies

Reference the cookie policy (or embed a summary). See `agency-cookie-consent` skill for full detail.

```
We use cookies and similar tracking technologies. For detailed information 
see our [Cookie Policy] / [Cookie Settings].
```

### 9. Children's Privacy

```
Our service is not directed to children under [13 / 16 — jurisdiction-dependent]. 
We do not knowingly collect personal information from children. If you believe 
we have inadvertently collected such data, contact us at [email] for immediate deletion.
```

- US: COPPA applies under age 13
- GDPR: member states may set the age between 13–16; default is 16

### 10. How to Contact Us / Lodge a Complaint

```
Data Privacy Contact: privacy@[yourdomain.com]

GDPR supervisory authority complaint: you have the right to lodge a complaint 
with the supervisory authority in your EU member state.
UK residents: Information Commissioner's Office (ICO) — ico.org.uk
```

---

## Phase 3 — GDPR-Specific Additional Requirements

If GDPR applies, also include:

- **Record of Processing Activities (RoPA)**: internal document (not in the public policy), required for organisations with > 250 employees or high-risk processing.
- **Data Protection Impact Assessment (DPIA)**: required for high-risk processing (large-scale processing of special categories, systematic profiling, public monitoring).
- **DPO appointment**: required if public authority, large-scale systematic monitoring, or large-scale processing of special category data.

---

## Phase 4 — Privacy Policy Review Checklist

Before publishing:

- [ ] Identity and contact details of data controller present
- [ ] DPO contact listed if applicable
- [ ] All categories of data collected are enumerated
- [ ] Legal basis stated for each processing purpose (GDPR)
- [ ] Retention periods specified for each category (not just "as long as necessary")
- [ ] All third-party processors/recipients named
- [ ] International transfer mechanism stated if data leaves EU/EEA
- [ ] All applicable consumer rights listed with exercise mechanism
- [ ] Cookie reference present (links to cookie policy or consent banner)
- [ ] Children's privacy statement present
- [ ] Supervisory authority complaint right mentioned (GDPR)
- [ ] "Last updated" date on the document
- [ ] Plain language — no legal jargon that obscures meaning
- [ ] Reviewed by qualified legal counsel

---

## Non-Negotiable Standards

- Never publish a privacy policy without a "last updated" date.
- Never use pre-generated templates verbatim — they must reflect the actual data flows of the specific product.
- Always audit actual third-party integrations in the codebase; developers often add analytics, error monitoring, or chat tools without updating the privacy policy.
- GDPR legal basis must be specific — "legitimate interests" requires a balancing test, "consent" requires a genuine opt-in mechanism.
- Always flag when a client's analytics setup (Google Analytics, Meta Pixel) requires a consent management platform (CMP) under GDPR.
