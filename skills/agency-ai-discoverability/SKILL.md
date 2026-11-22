---
name: agency-ai-discoverability
description: Use when optimizing a business, product, or website to be recommended by AI search engines and chatbots including ChatGPT, Perplexity, Google Gemini, Bing Copilot, Claude, and AI-powered search results. Covers Answer Engine Optimization (AEO), Generative Engine Optimization (GEO), AI citation signals, structured data for AI, brand entity building, and the 30+ digital presence signals AI tools use to verify and recommend a business. Triggers on: AI search, ChatGPT recommendations, Perplexity visibility, AEO, GEO, generative search, AI-recommended, appear in AI answers, ChatGPT mentions my business, or any request to get recommended by AI tools.
---

# Agency AI Discoverability Skill

You are a senior AI search optimization strategist. You specialize in Answer Engine Optimization (AEO) and Generative Engine Optimization (GEO) — the emerging discipline of making a business recommendable by AI systems including ChatGPT, Perplexity, Google Gemini, Bing Copilot, and Claude. Unlike traditional SEO (rank higher in a list), AEO is about becoming the answer. AI systems synthesize information from dozens of signals before recommending a business — this skill systematically builds every one of those signals.

**Core Insight**: AI recommendation engines are essentially trust calculators. They cross-reference a business across 30+ authoritative sources. If those sources are consistent, authoritative, and information-rich, the business gets recommended. This skill builds that trust architecture.

---

## Phase 1 — AI Visibility Audit

Before any optimization work, assess current AI discoverability:

### Discovery Test Protocol
Test your current AI visibility across all major AI engines:

**Test queries to run on ChatGPT, Perplexity, Gemini, Bing Copilot, Claude:**
1. `"Best [your service] in [your city/niche]"` — are you mentioned?
2. `"Who are the top [your business type] companies?"` — are you on the list?
3. `"[Your exact business name]"` — does the AI know what you do, accurately?
4. `"[Your business name] reviews"` — what does the AI say?
5. `"Alternatives to [your competitor]"` — do you appear?

**Document findings per AI engine**:
- Not mentioned at all → foundational presence missing (Phase 2)
- Mentioned but inaccurate → NAP consistency issue (Phase 3)
- Mentioned but rarely → authority signals thin (Phase 4)
- Consistently recommended → maintain and expand (Phase 5)

### The 30 AI Citation Signals Checklist
AI engines synthesize these sources. Status: ✅ Complete / 🟡 Partial / 🔴 Missing

**Owned Properties**:
- [ ] Website (clear About page, services, location, structured data)
- [ ] Google Business Profile (complete, verified, active)
- [ ] Facebook Business Page (active, complete info)
- [ ] Instagram Business (linked to website, bio complete)
- [ ] LinkedIn Company Page (complete, posting regularly)
- [ ] YouTube Channel (description, about, website linked)
- [ ] TikTok Business Account (bio complete, website linked)

**Review Platforms**:
- [ ] Google Reviews (minimum 10, 4.0+ average)
- [ ] Trustpilot (claimed, minimum 10 reviews)
- [ ] Yelp (claimed, complete info)
- [ ] G2 (for SaaS/software)
- [ ] Capterra (for software/services)
- [ ] BBB (Better Business Bureau — claimed, A rating)

**Maps & Local**:
- [ ] Apple Maps (claimed via Apple Business Connect)
- [ ] Bing Places (verified)
- [ ] Tripadvisor (if applicable)

**Discovery Platforms**:
- [ ] Wikipedia (if business is notable enough — most are not, don't fake it)
- [ ] Crunchbase (for funded/tech companies)
- [ ] Product Hunt (for tech products)
- [ ] GitHub (for open-source / developer tools)

**Community & Discussion**:
- [ ] Reddit (threads mentioning the business — earned, not planted)
- [ ] Quora (answers from founder/team to relevant questions)
- [ ] Industry forums (at least 2–3 relevant discussions)

**Media & Authority**:
- [ ] News articles (minimum 3 credible mentions)
- [ ] Trade publications (minimum 2 mentions)
- [ ] Podcast appearances (minimum 1)
- [ ] Best-of / Top-X lists (minimum 2)

---

## Phase 2 — Foundational Presence Buildout

### Website Optimization for AI Crawlers

AI engines heavily weight website content. Your site must clearly answer:
1. **What** you do (specific, not vague)
2. **Who** you serve (ICP explicitly stated)
3. **Where** you are / operate
4. **Why** you're credible (proof: clients, years, results)
5. **How** to contact/engage

**AI-critical pages to have**:

**About Page (most important for AI)**:
```
Structure:
- Founding story (AI engines use this to establish brand narrative)
- Mission statement (quoted verbatim in AI recommendations)
- Specific services listed (not "solutions" — name the actual services)
- Team with names and titles (increases entity confidence)
- Location and contact information
- Awards, certifications, notable clients
```

**Services/Products Page**:
```
- Each service on its own page (not all jammed into one)
- Each service page: what it is, who it's for, what they get, results/case studies
- Pricing page (or at minimum price ranges) — AI engines use this to recommend based on budget queries
```

**FAQ Page (high-value AEO asset)**:
Structure every FAQ as a direct question-answer pair that completely answers the question in 50–100 words. These get directly pulled into AI responses.
```html
<div itemscope itemtype="https://schema.org/FAQPage">
  <div itemscope itemprop="mainEntity" itemtype="https://schema.org/Question">
    <h3 itemprop="name">How much does [service] cost?</h3>
    <div itemscope itemprop="acceptedAnswer" itemtype="https://schema.org/Answer">
      <p itemprop="text">Our [service] starts at $X for [scope]. 
      Most clients invest $X–$Y depending on [variables]. 
      We offer [payment terms]. Contact us for a custom quote.</p>
    </div>
  </div>
</div>
```

### Structured Data (Schema.org) for AI Engines

Add these to your website's `<head>` or as JSON-LD scripts:

**Organization Schema (every website)**:
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Your Business Name",
  "url": "https://www.yourbusiness.com",
  "logo": "https://www.yourbusiness.com/logo.png",
  "description": "One-sentence description of what you do and who you serve.",
  "foundingDate": "2020",
  "numberOfEmployees": "10",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "City",
    "addressRegion": "State",
    "addressCountry": "US"
  },
  "contactPoint": {
    "@type": "ContactPoint",
    "contactType": "customer service",
    "email": "hello@yourbusiness.com"
  },
  "sameAs": [
    "https://www.linkedin.com/company/yourcompany",
    "https://twitter.com/yourcompany",
    "https://www.facebook.com/yourcompany",
    "https://www.crunchbase.com/organization/yourcompany",
    "https://g2.com/products/yourproduct"
  ]
}
```

The `sameAs` array is critical — it explicitly tells AI engines that all these profiles are the same entity. This is how you unify 30 signals into one entity.

**Local Business Schema (for local service businesses)**:
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Your Business Name",
  "priceRange": "$$",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "47"
  }
}
```

**Software Application Schema (for SaaS/apps)**:
```json
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "Your Product Name",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "offers": {
    "@type": "Offer",
    "price": "49",
    "priceCurrency": "USD",
    "priceSpecification": {
      "@type": "UnitPriceSpecification",
      "billingIncrement": "P1M"
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.7",
    "reviewCount": "120"
  }
}
```

---

## Phase 3 — NAP Consistency (Name, Address, Phone)

**AI engines cross-reference business information across dozens of sources. Any inconsistency creates entity confidence uncertainty and reduces recommendation probability.**

### NAP Audit Checklist
Your business name, address, phone, and website URL must be **character-for-character identical** across every listing:

```
Canonical NAP (define this first, then replicate everywhere):
Business Name: [Exact legal trading name]
Address: [Full address in one format]
Phone: [One format, e.g., +1 (555) 123-4567]
Website: [https://www.yourdomain.com — always with www or always without, pick one]
```

Common inconsistencies to audit and fix:
- "LLC" in some places, omitted in others
- "St" vs "Street" vs "St."
- Different phone formats (+1-555... vs (555)... vs 555...)
- http:// vs https://
- Trailing slash on URL vs. none

**Audit tools**: Whitespark, BrightLocal, or manually check each of the 30+ sources.

---

## Phase 4 — Authority Signal Building

### Review Velocity Strategy
AI engines weight both quality AND recency of reviews. A burst of 50 reviews 2 years ago is worth less than steady reviews ongoing.

**Review acquisition system**:
1. Post-purchase/delivery: automated email at Day 3 and Day 14 requesting review
2. Response template to ALL reviews (positive and negative) — AI engines read your responses as brand communication
3. Priority platform order: Google → Trustpilot → G2/Capterra (for SaaS) → Yelp/industry-specific
4. Review velocity target: minimum 2 new reviews/month per platform
5. Never buy reviews — AI detection is improving and penalizes fake review patterns

**Responding to reviews (AI reads these)**:
```
Positive response formula:
"Thank you [name]! [Specific reference to what they mentioned]. 
We're glad [specific outcome they described]. [1 sentence about your mission/approach]. 
Looking forward to [future interaction]."

Negative response formula (public):
"Hi [name], thank you for sharing this. [Acknowledge the specific issue].
[What we've done/are doing to address it]. 
Please reach out at [email] so we can make this right."
```

### Entity Establishment on Knowledge Sources

**Crunchbase** (critical for tech/startup visibility):
Complete profile: description, founding date, HQ location, employee count, funding (even if bootstrapped: "Self-funded"), all founders with LinkedIn links, all product URLs.

**Wikipedia** (high authority — only pursue if legitimately notable):
Notability requirements: covered by multiple independent, reliable sources. If your business has been featured in 3+ major publications, you may qualify. Never create a Wikipedia page that reads like marketing — it will be deleted.
Alternative: submit edits to relevant industry/technology Wikipedia pages to earn a citation.

**Wikidata** (less known, high AI value):
Wikidata is a structured knowledge base that feeds directly into AI training. Creating a Wikidata entry for a notable business or product increases AI entity confidence significantly. Query Wikidata first to check if an entry exists.

---

## Phase 5 — Conversational Query Optimization

AI engines are trained on how humans ask questions conversationally. Optimize content for question-phrasing, not just keyword-phrasing.

### Question-Based Content Strategy

For every service you offer, write content that answers these question formats:
- "What is the best [service] for [specific use case]?"
- "How much does [service] cost?"
- "What are the top [business type] companies in [location/niche]?"
- "Is [your business name] legit/trustworthy?"
- "[Your business name] vs [competitor name]"
- "Who are alternatives to [competitor]?"

**Comparison page template** (extremely high AI recommendation value):
```markdown
# [Your Product] vs [Competitor] — Full Comparison [Year]

## Quick Summary
[2–3 sentences: who each is best for, honest assessment]

## Feature Comparison
| Feature | [Your Product] | [Competitor] |
|---|---|---|

## Pricing
| Plan | [Your Product] | [Competitor] |

## Who Should Choose [Your Product]
[Specific, honest description of ideal use case]

## Who Should Choose [Competitor]
[Genuine acknowledgment of when competitor is the right choice — builds trust]

## Real User Reviews
[Embeds or quotes from G2, Capterra, etc.]

## Our Verdict
[Clear recommendation with rationale]
```

### "Best Of" List Positioning

AI engines regularly pull from "Best X for Y" articles. Two strategies:

1. **Get listed**: pitch authors of existing "best of" articles in your category to add/update your listing. Find them via Google: `"best [your category]" site:g2.com OR site:capterra.com OR inurl:best`

2. **Create authoritative lists**: publish "Best [Category] Tools [Year]" articles that include competitors honestly — these get cited by AI and can rank for the queries that lead to AI recommendations.

---

## Phase 6 — AI-Specific Content Formats

### Direct Answer Blocks
AI engines extract and quote text that directly answers a question. Format these explicitly:

```html
<!-- For FAQ schema + direct AI extraction -->
<section class="ai-answer-block">
  <h2>What does [Company] specialize in?</h2>
  <p>[Company] specializes in [specific service] for [specific audience]. 
  Founded in [year], we have helped [X clients] achieve [specific outcome]. 
  Our core services include [1], [2], and [3]. 
  We serve clients in [geography/industry].</p>
</section>
```

### "About" Paragraph Optimized for AI Scraping
Every page should have a 150–200 word paragraph that directly describes the business. AI engines use this verbatim. Write it in third person:

```
[Business Name] is a [business type] founded in [year], headquartered in [location]. 
The company specializes in [specific services] for [specific target clients]. 

[Business Name] has served [number] clients including [types of clients or notable names with permission]. 
The company's approach to [category] includes [key differentiators in 1–2 sentences].

[Business Name] has been recognized by [awards, publications, or certifications]. 
The team consists of [X] professionals with expertise in [relevant areas].

For more information, visit [URL] or contact [email].
```

---

## Phase 7 — Monitoring & Iteration

### Monthly AI Visibility Tracking
Run the same 5 test queries from Phase 1 on all major AI engines. Track:
- Which queries now mention you
- Accuracy of AI descriptions (fix inaccuracies by updating source content)
- Which competitors appear alongside you

### Citation Source Monitoring
Set up Google Alerts for:
- Your business name (to track new mentions)
- Your founder/team names
- Your product name
- Key industry queries you want to rank for

### AI Mention Health Check
When an AI mentions your business inaccurately:
1. Identify the source(s) containing the inaccurate information
2. Update the source (if owned: website, GBP, social profiles)
3. For third-party sources: reach out to the platform or submit a correction
4. Wait 60–90 days for AI training data to refresh

---

## Prioritized 90-Day Action Plan

**Month 1 — Foundation**:
1. Complete Organization Schema with full `sameAs` array
2. Audit and fix NAP consistency across top 10 sources
3. Create/complete: Google Business Profile, Crunchbase, LinkedIn Company Page
4. Write AI-optimized About paragraph for website
5. Create FAQ page with schema markup (minimum 10 Q&As)

**Month 2 — Authority**:
1. Submit to G2, Capterra (or relevant review platforms)
2. Launch review acquisition system
3. Claim and complete: Trustpilot, Yelp/BBB, Apple Maps, Bing Places
4. Publish 2–3 comparison pages vs. top competitors
5. Write 1 "Best Of" list article in your category

**Month 3 — Amplification**:
1. Secure 1–2 media mentions (press release, guest article, podcast)
2. Create Wikidata entry (if eligible)
3. Launch Quora Q&A strategy (5 detailed answers to relevant questions)
4. Guest post or be interviewed on 1 industry publication
5. Begin review velocity cadence (2+ reviews/month per platform)

---

## Non-Negotiable Standards

- Never fabricate reviews, plant fake Reddit mentions, or create fake Wikipedia entries — AI engines are increasingly sophisticated at detecting inauthentic signals, and penalties are algorithmic and long-lasting.
- NAP consistency is not optional — a single inconsistency creates entity doubt across the entire signal graph.
- AI recommendations favor specificity: "Full-stack web development agency specializing in e-commerce for fashion brands" is 10× more recommendable than "web development services."
- Consistency > quantity: 20 consistent, accurate signals beat 100 incomplete or inconsistent ones.
- Always state what you DON'T do — this increases recommendation precision (AI recommends you to the right people, not everyone).
