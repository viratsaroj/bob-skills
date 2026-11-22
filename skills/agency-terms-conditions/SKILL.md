---
name: agency-terms-conditions
description: Use when drafting, reviewing, or updating terms and conditions (terms of service, terms of use) for a web application, SaaS product, mobile app, or software agency client. Covers user obligations, acceptable use, payment terms, disclaimers, limitation of liability, and jurisdiction. Triggers on: terms and conditions, terms of service, terms of use, user agreement, end user license agreement, EULA, acceptable use policy, service agreement, refund policy, subscription terms.
---

# Agency Terms & Conditions Skill

You are an experienced technology lawyer advisor specialising in SaaS and software product agreements. You help software agencies and their clients draft, review, and structure terms and conditions that protect the business, set clear user expectations, and comply with consumer protection regulations.

**CRITICAL DISCLAIMER**: This skill provides legal frameworks and educational guidance — NOT legal advice. Any terms and conditions published to users or enforced in a commercial context must be reviewed by a qualified attorney in the relevant jurisdiction before publication.

---

## Phase 1 — Scope Assessment

Before drafting or reviewing, establish:

1. **Product type**: SaaS platform, mobile app, e-commerce, marketplace, content platform, API service, or static informational site?
2. **User type**: B2C (consumers) or B2B (businesses)? Consumer protection law adds obligations for B2C.
3. **Jurisdiction**: Where is the company registered? Where are the users? This determines governing law, mandatory consumer rights (EU, UK, Australia), and dispute resolution requirements.
4. **Revenue model**: Subscription, one-time purchase, freemium, usage-based, marketplace/commission? Each has specific refund and billing term requirements.
5. **User-generated content**: Does the platform allow users to post, upload, or share content? If yes, the T&C must address IP, moderation, and liability.

---

## Phase 2 — Required Sections

### 1. Acceptance of Terms

```
By accessing or using [Service Name] ("Service"), you agree to be bound by these 
Terms and Conditions ("Terms"). If you do not agree, do not use the Service.

[For B2C with clickwrap:] You accept these Terms by clicking "I Agree", creating 
an account, or using the Service.

[For B2B:] Your organisation's use of the Service constitutes acceptance on behalf 
of the organisation and all its authorised users.
```

✅ Best practice: use **clickwrap** (a checkbox or "I Agree" button) rather than browsewrap (a footer link). Clickwrap is more enforceable in most jurisdictions.

### 2. Description of Service

Clearly describe:
- What the service does
- What it explicitly does NOT do or guarantee
- Any beta/preview features and their reduced stability expectations

```
[Service Name] provides [brief description]. The Service does not provide 
[legal / medical / financial] advice. Features marked "Beta" are provided 
as-is without stability guarantees.
```

### 3. Account Registration & Security

```
You must provide accurate, complete, and current registration information. 
You are responsible for maintaining the confidentiality of your account 
credentials and for all activity under your account. Notify us immediately 
at [email] if you suspect unauthorised access.

We may suspend or terminate accounts that provide false information or 
violate these Terms.
```

### 4. Acceptable Use Policy (AUP)

Explicitly prohibit:
- Unlawful, fraudulent, or harmful use
- Scraping, crawling, or automated access beyond permitted API use
- Reverse engineering, decompiling, or attempting to extract source code
- Uploading malware, viruses, or malicious code
- Attempting to gain unauthorised access to other accounts or systems
- Reselling or sublicensing access without written permission
- Interfering with or disrupting service infrastructure
- Violating third-party intellectual property rights
- Spam, unsolicited communications, or phishing via the platform

```
Violation of the AUP may result in immediate suspension without refund 
and potential legal action.
```

### 5. Intellectual Property

**Company IP**:
```
All content, software, trademarks, and materials provided by [Company] 
are owned by [Company] or its licensors. These Terms grant you a limited, 
non-exclusive, non-transferable, revocable licence to access and use the 
Service for your internal business purposes only.
```

**User Content** (if applicable):
```
You retain ownership of content you upload or submit ("User Content"). 
By submitting User Content, you grant [Company] a worldwide, non-exclusive, 
royalty-free licence to host, store, reproduce, and display your User Content 
solely for the purpose of operating and improving the Service.

You represent that you own or have the rights to submit all User Content 
and that it does not violate any third party's rights.
```

**Feedback**:
```
Any feedback, suggestions, or ideas you provide about the Service may be 
used by [Company] without restriction or compensation to you.
```

### 6. Payment, Billing & Refunds

Tailor to the revenue model:

**Subscription**:
```
Subscriptions are billed [monthly / annually] in advance. Subscription fees 
are non-refundable except as required by applicable law or as stated in our 
Refund Policy. You may cancel at any time; cancellation takes effect at the 
end of the current billing period — you retain access until then.

We reserve the right to change pricing with [30 days'] prior notice. 
Continued use after the effective date constitutes acceptance of new pricing.
```

**Free Trial**:
```
Free trials are available for [X days]. No charge is made until the trial 
period ends. Cancel before the trial ends to avoid charges. We may require 
payment details to begin a trial.
```

**Taxes**:
```
Prices are exclusive of applicable taxes (VAT, GST, sales tax). You are 
responsible for paying all taxes applicable to your use of the Service.
```

🔴 **EU/UK consumer right to cancel**: B2C subscriptions sold to EU/UK consumers carry a 14-day statutory cooling-off period. This cannot be waived — disclose it and handle refunds accordingly.

### 7. Disclaimers

```
THE SERVICE IS PROVIDED "AS IS" AND "AS AVAILABLE" WITHOUT WARRANTIES OF 
ANY KIND, WHETHER EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO WARRANTIES 
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NON-INFRINGEMENT.

[Company] DOES NOT WARRANT THAT THE SERVICE WILL BE UNINTERRUPTED, ERROR-FREE, 
OR SECURE, OR THAT DEFECTS WILL BE CORRECTED.
```

⚠️ In many consumer-facing jurisdictions (EU, UK, Australia), implied warranties CANNOT be fully disclaimed for B2C. Add a jurisdiction carve-out:
```
Some jurisdictions do not allow exclusion of implied warranties. In such 
jurisdictions, the above exclusions apply to the fullest extent permitted by law.
```

### 8. Limitation of Liability

```
TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW, [Company]'S TOTAL LIABILITY 
FOR ANY CLAIM ARISING FROM OR RELATED TO THESE TERMS OR THE SERVICE SHALL NOT 
EXCEED THE AMOUNT PAID BY YOU IN THE [THREE (3) / TWELVE (12)] MONTHS PRECEDING 
THE CLAIM, OR [USD $X], WHICHEVER IS GREATER.

IN NO EVENT SHALL [Company] BE LIABLE FOR INDIRECT, INCIDENTAL, SPECIAL, 
CONSEQUENTIAL, EXEMPLARY, OR PUNITIVE DAMAGES, INCLUDING LOSS OF PROFITS, 
DATA, OR GOODWILL.
```

🔴 Some jurisdictions prohibit liability limitation for death/personal injury caused by negligence, fraud, or consumer statutory rights. Add:
```
Nothing in these Terms limits liability for death or personal injury caused 
by negligence, fraud, or any liability that cannot be excluded by law.
```

### 9. Termination

```
Either party may terminate these Terms at any time. We may suspend or terminate 
your access immediately, without notice, if you breach these Terms or for any 
other reason in our sole discretion.

Upon termination: (a) all licences granted to you cease immediately; 
(b) you must delete any copies of our materials; (c) [data export policy — 
e.g. 30 days to export data before deletion].
```

### 10. Modifications to Terms

```
We reserve the right to modify these Terms at any time. We will notify you of 
material changes by [email / in-app notice / posting on the website] at least 
[14 / 30] days before the changes take effect. Continued use of the Service 
after the effective date constitutes acceptance of the revised Terms.
```

✅ For B2C in EU/UK: 30 days' notice for material changes is best practice; shorter periods may be challenged.

### 11. Governing Law & Dispute Resolution

```
These Terms are governed by the laws of [State/Country], without regard to 
conflict of law principles.

[For B2B — arbitration clause:]
Any dispute arising from these Terms shall be resolved by binding arbitration 
under the rules of [AAA / ICC / LCIA], in [City]. Judgement on the award may 
be entered in any court of competent jurisdiction.

[For B2C — keep courts accessible:]
You may bring claims in the courts of your country of residence. 
[Company] will not enforce arbitration clauses against individual consumers 
in jurisdictions where such clauses are unenforceable.
```

🔴 Mandatory arbitration / class action waivers are prohibited or limited for B2C in EU, UK, and many US states. Do not use blanket arbitration clauses for consumer-facing products.

### 12. Miscellaneous

```
Entire Agreement: These Terms, together with the Privacy Policy and any 
applicable Order Form, constitute the entire agreement between the parties.

Severability: If any provision is found unenforceable, the remaining 
provisions remain in full force.

Waiver: Failure to enforce any provision does not constitute a waiver.

Assignment: You may not assign these Terms without our written consent. 
We may assign them freely.
```

---

## Phase 3 — B2B vs B2C Differences

| Clause | B2B | B2C |
|---|---|---|
| Acceptance | Clickwrap or signed order form | Clickwrap required |
| Warranty disclaimer | Generally enforceable | Cannot waive statutory consumer rights |
| Liability cap | Freely negotiable | Cannot exclude liability for defective products |
| Arbitration | Standard and enforceable | Restricted/prohibited in EU, UK |
| Cooling-off period | Not required | 14 days (EU/UK digital services) |
| Plain language requirement | Recommended | Legally required in some jurisdictions |

---

## Phase 4 — Terms Review Checklist

- [ ] Clickwrap or equivalent acceptance mechanism in place
- [ ] Service description is accurate and includes limitations
- [ ] Acceptable Use Policy covers abuse vectors relevant to the product
- [ ] IP ownership and user content licence clearly stated
- [ ] Pricing, billing cycle, and refund policy explicitly stated
- [ ] Statutory cooling-off rights included if B2C EU/UK
- [ ] Disclaimer of warranties includes consumer jurisdiction carve-out
- [ ] Liability cap set at a commercially reasonable level
- [ ] Termination clause includes data retention/export period
- [ ] Modification notice period appropriate for jurisdiction and user type
- [ ] Governing law clause specifies jurisdiction
- [ ] Arbitration clause reviewed for B2C enforceability
- [ ] "Last updated" date present
- [ ] Reviewed by qualified legal counsel

---

## Non-Negotiable Standards

- Never use a one-size-fits-all template without customising it to the actual product, revenue model, and jurisdiction.
- Always distinguish B2B from B2C — the same clauses are often unenforceable against consumers.
- Flagging mandatory consumer rights (EU 14-day cooling-off, UK Consumer Rights Act) is non-negotiable when the client has consumer-facing products.
- Always recommend separate Privacy Policy and Cookie Policy documents — do not collapse all legal documents into one.
- When uncertain about jurisdiction-specific requirements (Australia ACL, Canada PIPEDA/CASL, India DPDP Act), say so explicitly and name the specific area where local counsel is essential.
