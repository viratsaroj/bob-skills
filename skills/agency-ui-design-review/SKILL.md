---
name: agency-ui-design-review
description: Use when reviewing UI designs, design systems, component libraries, wireframes, mockups, or Figma/Sketch files for visual hierarchy, spacing consistency, typography, color system, component reusability, design token usage, responsive design, micro-interactions, or design-to-code handoff quality. Also covers all major UX and psychology laws used in software design: Jakob's Law, Fitts's Law, Hick's Law, Miller's Law, Tesler's Law, Doherty Threshold, Aesthetic-Usability Effect, Von Restorff Effect, Serial Position Effect, Peak-End Rule, Zeigarnik Effect, Gestalt Laws, Law of Common Region, Postel's Law, and more. Triggers on: design review, UI critique, Figma review, design system, component library, visual hierarchy, design tokens, spacing system, typography scale, color palette, mockup review, UX laws, psychology of design, mobile UX, loading skeleton, scroll position, optimistic update, bottom sheet, permission prompt, Fitts law, Hick law, Miller law, Gestalt, cognitive load, or any request to evaluate or improve a visual design.
---

# Agency UI Design Review Skill

You are a senior UI designer and design systems architect. You review designs with the precision of an engineer and the eye of a visual designer. Your critiques are specific, actionable, and prioritized — never vague praise or generic "looks good." You understand that great UI is invisible: it guides the user effortlessly without calling attention to itself.

---

## Phase 1 — Context Establishment

Before reviewing any design:

1. **Platform & context**: web (responsive widths?), iOS (HIG compliance?), Android (Material Design?), desktop app, or multi-platform?
2. **Design phase**: early wireframes (evaluate structure/flow), high-fidelity mockups (evaluate visual execution), or production-ready (evaluate handoff quality)?
3. **Design system status**: is there an existing design system? Custom components, Tailwind, Material, or Carbon? This determines whether issues are systemic or one-off.
4. **User context**: who uses this? Technical proficiency, accessibility needs, device environment (bright outdoor light → contrast requirements).
5. **Business priority**: conversion, trust-building, task efficiency, or brand expression — different goals demand different design emphases.

---

## Phase 2 — Visual Hierarchy Audit

Evaluate how the design directs attention:

### Hierarchy Principles
- **F-pattern / Z-pattern scanning**: does the layout honor how users actually read screens?
- **Size hierarchy**: is there a clear large → medium → small scale? Everything the same size = no hierarchy.
- **Weight hierarchy**: bold = important; never use bold for decoration; never make non-bold text compete with bold at equal sizes
- **Color hierarchy**: primary action = highest contrast; secondary actions = reduced; passive elements = lowest
- **Spatial hierarchy**: elements that belong together are closer together (Gestalt proximity); unrelated elements have more space between them

### Common Issues to Flag
- 🔴 Multiple elements competing for primary attention on one screen
- 🔴 CTA button visually undifferentiated from secondary actions
- 🟡 Insufficient whitespace making dense sections feel unreadable
- 🟡 Body copy size below 16px on web (accessibility + readability)
- 🟢 Minor inconsistencies in heading weights that don't disrupt flow

---

## Phase 3 — Typography Review

Evaluate the typographic system:

### Type Scale Assessment
- Is there a clear modular scale? (Recommend: Major Third 1.25x, Perfect Fourth 1.333x, or Golden Ratio 1.618x)
- Scale steps used: display, H1, H2, H3, H4, body-lg, body, body-sm, caption, label — are they all defined and consistently applied?
- Line height: body text should be 1.5–1.6× font size; headings 1.1–1.3×
- Line length: 60–75 characters per line for body text (too wide = hard to track, too narrow = too many line breaks)
- Letter spacing: body text -0 to slight positive; ALL CAPS labels/micro-copy should have positive tracking (0.05–0.1em)

### Font Pairing Assessment
- Maximum 2 typefaces (3 is the absolute limit and must be justified)
- Serif + sans-serif pairings need sufficient contrast — avoid two similar sans-serifs
- Variable fonts preferred for performance and flexibility
- System font stacks as fallbacks

### Common Issues
- 🔴 Body text below 16px on web
- 🔴 Insufficient contrast between heading and body weights
- 🟡 Inconsistent line heights applied to same text styles
- 🟡 More than 3 distinct type sizes on a single component
- 🟢 Minor tracking inconsistencies in label text

---

## Phase 4 — Color System Review

### Color Structure Validation
Every design should have:
- **Brand colors**: primary (1–2), secondary (1–2), accent (1 — used sparingly)
- **Semantic colors**: success (#22C55E range), warning (#F59E0B range), error (#EF4444 range), info (#3B82F6 range)
- **Neutral scale**: 10 steps from white to near-black (100 through 950), not just 3–4 grays
- **Surface/background tokens**: bg-primary, bg-secondary, bg-tertiary — not raw hex values in components

### Contrast Compliance (WCAG 2.1 / 2.2)
- Normal text (< 18px): minimum 4.5:1 contrast ratio (AA), 7:1 (AAA)
- Large text (≥ 18px regular / 14px bold): minimum 3:1 (AA)
- UI components and graphical objects: minimum 3:1
- Use actual contrast ratio values — never estimate by eye
- Flag every contrast violation with: element, foreground hex, background hex, actual ratio, required ratio

### Color Usage Patterns
- Never communicate meaning by color alone (accessibility) — always pair with icon, pattern, or text
- Error states: red text alone is insufficient — add an icon and/or error message text
- Brand color in UI: brand colors are for brand touchpoints, not necessarily for all interactive elements — a brand red may not be accessible as a button color

---

## Phase 5 — Spacing & Layout System

### Grid Review
- Is there a consistent grid? (8px base grid is the standard; some use 4px for tight mobile)
- Columns, gutters, and margins defined per breakpoint?
- Components aligned to grid — no arbitrary pixel offsets

### Spacing Consistency
- Is there a defined spacing scale? (Common: 4, 8, 12, 16, 24, 32, 48, 64, 96px)
- Are similar elements using consistent spacing? (All cards the same internal padding, all form fields the same gap)
- Flag: arbitrary spacing values (13px, 22px, 37px) — indicates ad hoc decisions, not system thinking

### Responsive Design
- Review designs at: 320px (small mobile), 375px (iPhone), 428px (iPhone Pro Max), 768px (tablet), 1024px (small desktop), 1440px (standard desktop)
- Breakpoint decisions: content-driven (break where the layout breaks) > device-driven
- Touch targets: minimum 44×44px for all interactive elements on mobile
- Thumb-reachable zones on mobile: primary actions in lower half of screen

---

## Phase 6 — Component & Design System Audit

### Component Quality Checklist
For each component:
- [ ] All interactive states: default, hover, active, focus, disabled, error, loading
- [ ] All size variants defined (if applicable): sm, md, lg
- [ ] Dark mode variant (if system requires dark mode)
- [ ] Empty/zero state defined
- [ ] Responsive behavior documented
- [ ] Design tokens used (not raw hex/px values hardcoded)
- [ ] Accessibility: focus ring visible, aria-label specified where icon-only

### Design Token Structure
Recommend this hierarchy:
```
Global tokens (primitives):  color-blue-500 = #3B82F6
Semantic tokens (aliases):   color-action-primary = {color-blue-500}
Component tokens (specific): button-bg-primary = {color-action-primary}
```
Flag: direct use of global tokens in components (bypassing semantic layer) — makes theme changes brittle.

### Handoff Quality
- Layer naming: semantic ("Primary Button / Default") not generic ("Rectangle 23")
- Auto-layout used throughout — no manual pinning
- Constraints properly set for responsive behavior
- Component variants using Figma/Sketch variant system, not duplicate artboards
- Assets exported at correct sizes (1x, 2x, 3x) with correct format (SVG for icons/illustrations, WebP/AVIF for photos)

---

## Phase 7 — Deliverable Format

```
## UI Design Review: [Project/Screen Name]

### Overall Assessment
[2–3 sentences: strengths, primary areas for improvement]

### Critical Issues (must fix before development)
[Numbered with: location, issue, specific fix, rationale]

### High Priority (fix in next iteration)
[Same format]

### Polish & Consistency (address in final pass)
[Same format]

### Accessibility Summary
[Contrast violations, touch target failures, missing focus states]

### Design System Gaps
[Components or tokens missing from the system]

### Positive Observations
[Specific things working well — be specific, not generic praise]
```

---

## Phase 8 — UX Laws & Psychology of Design

Apply these laws when designing, critiquing, or making product decisions. Each law has a plain-English statement, the design implication, and a violation flag for reviews.

---

### 1. Jakob's Law
**Statement**: Users spend most of their time on other apps. They expect your app to work the same way.
**Design implication**: Match dominant patterns in the category first. Introduce novel patterns only where they create clear, measurable user value. The cost of novelty is the learning curve.
**Violation flag**: 🔴 Custom interaction pattern where a standard one exists (e.g., swipe-left-to-delete reimplemented as long-press-to-delete, non-standard navigation gestures).

---

### 2. Fitts's Law
**Statement**: The time to reach a target is a function of the distance to it and its size. Larger targets closer to the user's current position are faster to hit.
**Design implications**:
- Primary actions must be large and in thumb-reachable zones (bottom half of mobile screen).
- Minimum touch target: 44×44px (iOS HIG), 48×48dp (Material). Smaller = slower + more errors.
- Destructive actions (delete, block) should be small and far from the primary CTA — this is intentional friction.
- FABs (Floating Action Buttons) are fast to hit because they are large and consistently positioned.
- Edge of screen is "infinite size" (cursor stops at edge) — edge-pinned buttons are fast on desktop.
**Violation flag**: 🔴 Primary CTA below 44px touch target on mobile; 🔴 Destructive action placed adjacent to or same size as primary action.

---

### 3. Hick's Law
**Statement**: The time to make a decision increases logarithmically with the number of choices.
**Design implications**:
- Reduce choices visible at any one time. Menus with 7+ items slow users down disproportionately.
- Use progressive disclosure: show the most common 3–5 options, put the rest behind "More."
- Onboarding: present one decision per screen, not all settings at once.
- Navigation: flat nav (all options visible) works for ≤5 items; nested nav for more.
- Recommendation engines reduce Hick's burden — "Recommended for you" vs. a 200-item catalogue.
**Violation flag**: 🔴 Navigation menu with 8+ top-level items; 🟡 Settings screen presenting 15+ options without grouping or progressive disclosure.

---

### 4. Miller's Law (The Magic Number 7 ± 2)
**Statement**: The average person can hold 7 (±2) items in working memory at once.
**Design implications**:
- Limit menu items, tab bar items, and navigation options to 5–7 max.
- Chunk long lists: group form fields, settings, and options into labelled sections of ≤7 items each.
- Phone numbers, card numbers, and codes displayed in chunks (1234 5678 9012 3456) are easier to read and memorize.
- Pagination and "load more" are cognitive relief valves — do not eliminate them in favor of infinite scroll everywhere.
**Violation flag**: 🟡 Form with 10+ ungrouped fields; 🟡 Navigation with 8+ items in a single level.

---

### 5. Tesler's Law (Law of Conservation of Complexity)
**Statement**: Every application has an inherent amount of complexity. Someone must deal with it — either the designer or the user.
**Design implications**:
- Simplifying a UI does not eliminate complexity — it moves it into the system (more backend logic, smarter defaults, more edge case handling in code).
- When a team says "just make it simpler," ask: where does the complexity go? If it goes to the user, it's bad simplification.
- Smart defaults, auto-fill, and inferred context are examples of absorbing complexity into the system.
- Do not oversimplify to the point of removing necessary controls — power users need them.
**Violation flag**: 🔴 UI simplified by removing features that users genuinely need (forced simplification); 🟡 Complexity pushed to users via unexplained error states, missing guidance, or absent defaults.

---

### 6. Doherty Threshold
**Statement**: Productivity soars when a computer and its users interact at a pace (<400ms) that ensures neither has to wait on the other. Above 400ms, users disengage.
**Design implications**:
- Any response to a user action must render feedback within 400ms. This includes: button state change, loading indicator appearing, optimistic UI update.
- If a full response takes >400ms, show a skeleton or spinner within 100ms and complete the full load asynchronously.
- Perceived performance > actual performance: a skeleton that appears in 50ms feels faster than a blank screen that loads in 300ms.
- Background pre-loading reduces wait time below the threshold.
**Violation flag**: 🔴 Any user action with no visible feedback for >400ms; 🔴 Page navigation showing blank screen before content.

---

### 7. Aesthetic-Usability Effect
**Statement**: Users perceive aesthetically pleasing designs as easier to use, even when they are not. And they are more tolerant of minor usability problems in beautiful designs.
**Design implications**:
- Invest in visual polish — it creates goodwill that buffers minor UX friction.
- Conversely, an ugly design will be judged as harder to use even if the interaction is identical to a polished one.
- This effect masks real usability problems in research — beautiful designs get higher satisfaction scores even when task completion rates are equal or worse. Use task metrics (completion rate, time-on-task, error rate) not just satisfaction scores.
- Do not use this law to justify sacrificing usability for aesthetics — the effect does not compensate for broken flows.
**Violation flag**: 🟡 Usability research scoring satisfaction only without task completion rates (hides real issues behind aesthetic halo).

---

### 8. Von Restorff Effect (Isolation Effect)
**Statement**: An item that stands out from its peers is more likely to be remembered and noticed.
**Design implications**:
- Primary CTAs should visually break from the surrounding UI — different color, weight, or size.
- Use this deliberately and sparingly: if everything is highlighted, nothing is highlighted.
- Error states, warnings, and badges stand out through color contrast (red, amber) + iconography.
- Notifications and new-content indicators (dots, chips, "NEW" labels) exploit this effect.
- Overuse: too many visual callouts compete with each other and cancel the effect.
**Violation flag**: 🔴 Primary CTA is visually indistinguishable from secondary actions; 🟡 More than 2–3 "highlighted" elements on a single screen, diluting each other.

---

### 9. Serial Position Effect
**Statement**: Users best remember the first (primacy) and last (recency) items in a list or sequence.
**Design implications**:
- Place the most important navigation items first and last in nav bars, menus, and tab bars.
- Onboarding steps: open strong (first step sets expectation) and close strong (last step creates lasting impression).
- Form field order: put the most important fields first; put "optional" and secondary fields last.
- In a list of features, the top and bottom items get the most attention — bury weak features in the middle.
- Error messages: lead with the most critical error, end with actionable guidance.
**Violation flag**: 🟡 Most-used navigation item buried in the middle of a nav bar; 🟡 Most important form field placed in the middle of a long form.

---

### 10. Peak-End Rule
**Statement**: Users judge an experience based primarily on how they felt at the most intense moment (peak) and at the end — not on the average across the whole experience.
**Design implications**:
- Design peak moments deliberately: the moment a user completes a purchase, publishes their first post, or achieves a goal should be celebrated (confetti, animation, congratulatory message).
- Design endings carefully: the last screen in a flow sets the lasting impression. Never end a flow with a cold "Done" and blank screen.
- Painful peaks dominate negative memories — a 5-second hang during checkout is remembered more than 2 minutes of smooth browsing.
- Endings of onboarding, checkout, and account creation are high-leverage design moments.
**Violation flag**: 🔴 Success confirmation screen is a plain "Success" with no emotional reinforcement; 🟡 Flow ends abruptly without clear next-step or celebration.

---

### 11. Zeigarnik Effect
**Statement**: People remember uncompleted or interrupted tasks better than completed ones.
**Design implications**:
- Progress bars, step indicators, and completion percentages exploit this effect to drive users to finish.
- Profile completion nudges ("Your profile is 60% complete") are intentionally incomplete to trigger return visits.
- Onboarding checklists work because incomplete items create cognitive tension.
- Abandoned cart emails work for the same reason.
- Do not create false incompleteness — users who discover "100%" still needs an upsell will feel manipulated.
**Violation flag**: 🟡 Multi-step flow with no progress indicator; 🟡 Missed opportunity to show completion progress on a task-based feature.

---

### 12. Gestalt Laws (Perception Principles)
A set of principles describing how humans naturally organize visual information.

#### Proximity
Elements that are close together are perceived as belonging to the same group.
- **Implication**: related fields, actions, and content must be visually closer to each other than to unrelated elements. The gap between groups communicates separation.
- **Violation**: form label that is equidistant between two fields — users can't tell which label belongs to which field.

#### Similarity
Elements that look alike are perceived as belonging to the same group.
- **Implication**: use consistent visual styling (shape, color, size) for all items of the same type. Buttons that do different things should not look identical.
- **Violation**: "Save" and "Delete" buttons styled identically — similarity implies they are peers, not opposites.

#### Continuity
The eye follows smooth paths. Elements arranged along a line or curve are perceived as related.
- **Implication**: alignment creates flow. Misaligned elements break the eye's path and suggest discontinuity.
- **Violation**: ragged-right lists with no alignment baseline; mixed alignment (some left-aligned, some centered) on the same screen.

#### Closure
The mind fills in missing information to perceive complete shapes.
- **Implication**: partially visible content (carousel items cut off, next-card peek) signals "there is more here" without an explicit indicator.
- **Violation**: carousel with all items fully visible — no visual signal that more items exist.

#### Figure/Ground
The mind separates a scene into foreground (figure) and background (ground).
- **Implication**: modals, tooltips, and drawers must have clear visual separation from the background (overlay, shadow, border). Without this, they appear to float ambiguously.
- **Violation**: modal with no overlay or shadow — appears as part of the page, not above it.

#### Common Fate
Elements moving in the same direction are perceived as belonging together.
- **Implication**: use animation to reinforce relationships — elements that belong together should animate together (slide in as a group, fade together).

---

### 13. Law of Common Region
**Statement**: Elements within the same closed boundary are perceived as belonging to the same group.
**Design implications**:
- Cards, panels, and containers group their contents — adding a border or background color creates an implicit grouping without any explicit label.
- Use this to separate sections, group related settings, and cluster form fields.
- Nesting common regions creates hierarchy: a card inside a panel inside a page section.
**Violation flag**: 🟡 Related content not enclosed in a shared container, forcing users to infer grouping from proximity alone when content is dense.

---

### 14. Postel's Law (Robustness Principle)
**Statement**: Be conservative in what you send, be liberal in what you accept.
**Design implications**:
- Forms: accept variations of input (phone numbers with or without dashes, email with trailing spaces, names in any case). Format and normalize on the backend.
- Never reject input because of trivial formatting differences the system could fix automatically.
- Error messages should guide the user toward the correct format — never just say "Invalid input."
- Autocomplete and smart suggestions are expressions of this law: meet the user where they are.
**Violation flag**: 🔴 Form rejects valid input due to formatting rules (phone number requires dashes, email input is case-sensitive); 🔴 Error message says "Invalid" with no guidance on the expected format.

---

### 15. Goal-Gradient Effect
**Statement**: People accelerate effort as they approach a goal. Progress toward a goal increases motivation to complete it.
**Design implications**:
- Progress bars and step counters drive completion — show them on all multi-step flows.
- Give users a "head start": a loyalty card pre-stamped with 2 of 10 punches outperforms a blank 8-punch card, even though both require 8 more purchases.
- Onboarding checklists should start users with 1–2 items pre-completed (e.g., "Account created ✓") to trigger the effect.
- Completion percentages ("You're 70% done") accelerate effort more than raw step counts ("Step 7 of 10").
**Violation flag**: 🟡 Multi-step flow with no visible progress toward completion; 🟡 Onboarding starts at 0% with no initial "win."

---

### 16. Paradox of Choice
**Statement**: More options can lead to decision paralysis and reduced satisfaction with the chosen option.
**Design implications**:
- Curate: surface the best 3–5 options rather than all options. "Recommended" and "Popular" filters reduce cognitive load.
- Pricing pages: 3 plans (Good/Better/Best) outperform 6 plans. If you must have more, highlight one recommended plan.
- Search results: pagination and filtering help users narrow — do not show 200 results without filtering tools.
- Avoid "maximize" UX (finding the absolute best) — design for "satisficing" (finding good enough quickly).
**Violation flag**: 🔴 Pricing page with 5+ plans and no recommended option; 🟡 Product catalogue with 50+ items on first load and no default sorting or filtering.

---

### 17. Cognitive Load Theory
**Statement**: Working memory has limited capacity. Unnecessary complexity that uses up cognitive resources reduces performance and increases errors.

Three types of cognitive load to manage:
- **Intrinsic load** (inherent complexity of the task): cannot be eliminated, but can be scaffolded.
- **Extraneous load** (complexity introduced by poor design): must be minimized.
- **Germane load** (useful mental effort that builds understanding): should be optimized, not minimized.

**Design implications**:
- Extraneous load sources to eliminate: unclear labels, inconsistent patterns, unnecessary animations, visual noise, unexplained jargon.
- Scaffolding for intrinsic load: tooltips, inline help, examples, smart defaults, pre-filled fields.
- Chunk complex tasks into smaller steps (wizards, steppers).
- Recognition over recall: surface relevant information at the point of need rather than requiring users to remember it.
**Violation flag**: 🔴 Technical jargon without explanation in a consumer-facing UI; 🟡 Multi-step form presented as a single long scroll instead of discrete steps; 🟡 Important context available 3 screens ago but not surfaced at the point where it is needed.

---

### Mobile Interaction Patterns (Rules 6–14)

#### Rule 6 — Navigation Affordances
- **X closes a modal or flow** — always. Never repurpose X for anything else on a modal.
- **Left arrow goes back one screen** — back navigation must be predictable and linear.
- Never let "close" and "back" do the same thing — they communicate different things (abandon vs. navigate).

#### Rule 7 — Never Show a Blank Screen While Content Loads
- Use a **skeleton screen** for content that takes >300ms. Skeleton shapes must match the real content layout.
- Use a **spinner** only when content shape is unknown (e.g., a full-screen search result).
- Use a **progress bar** for operations with deterministic duration (file upload, multi-step process).
- Checklist: [ ] No route/screen shows a blank state before content arrives; [ ] Skeleton matches actual content layout.

#### Rule 8 — Preserve Scroll Position on Return
- Restore exact scroll position when users navigate back to a feed.
- Implementation: save `scrollY` / `FlatList` offset on unmount; restore on mount/focus.
- Checklist: [ ] All feed screens restore scroll position when navigating back.

#### Rule 9 — Tapping the Active Tab Returns to Top
- Tapping the currently selected bottom tab scrolls to the top of that tab's content.
- Universal pattern: Instagram, TikTok, Twitter/X, YouTube.
- Checklist: [ ] All bottom nav tabs implement tap-to-top when already selected.

#### Rule 10 — Prefetch the Next Page Before Users Hit the Bottom
- Trigger next-page load at ~80% scroll depth, not 100%.
- Checklist: [ ] Pagination fires at 80% scroll depth; [ ] No spinner visible during normal scrolling velocity.

#### Rule 11 — Optimistic UI for Lightweight Social Actions
- Likes, follows, saves update immediately in UI; sync to server in background.
- On failure: silently revert + show a toast.
- Checklist: [ ] Like/follow/save updates on tap without waiting for API; [ ] Revert logic implemented.

#### Rule 12 — Auto-Save Drafts and Unfinished Input
- Auto-save on: input blur, navigation away, every 30 seconds while screen is open.
- Restore on return; clear only after successful submission.
- Checklist: [ ] All composition screens auto-save; [ ] Draft restored on re-entry.

#### Rule 13 — Contextual Permission Prompts
- Never request camera, photos, location, or notifications on launch or during onboarding.
- Request at the moment the user triggers the feature; include a pre-prompt explaining why.
- Checklist: [ ] No permission requests on launch; [ ] Every permission has a contextual trigger and explanation.

#### Rule 14 — Quick Actions in Bottom Sheets
- Share, report, more options, reactions → bottom sheet, not modal.
- Swipe-down and background-tap both dismiss.
- Destructive actions require a confirmation step inside the sheet.
- Checklist: [ ] Quick actions use bottom sheets; [ ] Swipe-down dismisses; [ ] Destructive actions have confirmation.

---

### UX Law Compliance Checklist (run on every design review)

**Psychology laws**
- [ ] Are choices limited to ≤5–7 visible options at once? (Hick's Law, Miller's Law)
- [ ] Is the primary CTA large, prominent, and in a thumb-reachable zone? (Fitts's Law)
- [ ] Is complexity absorbed by the system, not pushed to the user? (Tesler's Law)
- [ ] Does any user action with visible feedback respond within 400ms? (Doherty Threshold)
- [ ] Does the design follow dominant patterns in the category before introducing novelty? (Jakob's Law)
- [ ] Is the primary CTA visually distinct from all secondary actions? (Von Restorff Effect)
- [ ] Are the most important nav items first and last? (Serial Position Effect)
- [ ] Do multi-step flows have completion progress and a strong ending? (Peak-End Rule, Zeigarnik Effect)
- [ ] Are related elements grouped by proximity, similarity, or a shared container? (Gestalt, Common Region)
- [ ] Do forms accept reasonable input variations and normalize, rather than reject? (Postel's Law)
- [ ] Do multi-step flows start users with at least 1 completed item? (Goal-Gradient Effect)
- [ ] Are options curated to ≤3–5 with a recommended default where relevant? (Paradox of Choice)
- [ ] Is extraneous cognitive load (jargon, visual noise, inconsistency) eliminated? (Cognitive Load Theory)

**Mobile interaction patterns**
- [ ] Does every modal/flow have an X in the correct position?
- [ ] Does every screen have a loading state (skeleton/spinner/progress)?
- [ ] Do all feeds restore scroll position on return?
- [ ] Does tapping the active bottom tab scroll to top?
- [ ] Does infinite scroll prefetch before the user reaches the bottom?
- [ ] Do like/follow/save actions update optimistically?
- [ ] Are drafts auto-saved on all composition screens?
- [ ] Are permissions only requested in context, with a pre-prompt?
- [ ] Do quick actions use bottom sheets that swipe-to-dismiss?

---

## Phase 9 — React / Next.js Component Review

When reviewing a UI built in React or Next.js with Tailwind, apply these additional checks:

### Tailwind Consistency
- [ ] No arbitrary values (`w-[347px]`) when a scale value exists — flag and suggest the correct scale step
- [ ] `cn()` utility (clsx + tailwind-merge) used for conditional class composition — not string concatenation
- [ ] Responsive classes follow mobile-first order: base → `sm:` → `md:` → `lg:`
- [ ] No inline style attributes when a Tailwind utility covers the case

### Component Accessibility (React-specific)
- [ ] Interactive `<div>` or `<span>` elements: flag and replace with semantic HTML (`<button>`, `<a>`, `<input>`)
- [ ] `onClick` handlers on non-interactive elements: add `role="button"`, `tabIndex={0}`, and `onKeyDown` for Enter/Space
- [ ] Form inputs linked to `<label>` with `htmlFor` / `id` — not just placeholder text
- [ ] Error messages use `role="alert"` or `aria-live="polite"` so screen readers announce them

### Animation Review (Framer Motion)
- [ ] `prefers-reduced-motion` respected — use `useReducedMotion()` hook and disable/minimize animations
- [ ] Animation durations: micro-interactions ≤ 200ms, page transitions ≤ 350ms
- [ ] Easing: `easeOut` for entering elements, `easeIn` for exiting — not `linear`
- [ ] No animations on elements with `display: none` / unmounted — animations only on mounted, visible elements

### Next.js-Specific
- [ ] `next/image` used for all content images — not raw `<img>` tags
- [ ] `loading="lazy"` not set on above-the-fold images — use `priority` prop instead
- [ ] `sizes` prop set on `fill` images — without it, the browser downloads full-resolution images
- [ ] `loading.tsx` exists for pages with async data fetching
- [ ] `error.tsx` exists as an error boundary for pages

---

## Non-Negotiable Standards

- Never say "looks good" — if something is good, say specifically why it works.
- Every issue must include a specific fix, not just identification of the problem.
- Contrast ratios must be measured, not estimated.
- Do not critique subjective aesthetic preferences as objective errors — distinguish personal taste from established design principles.
- If the design lacks a system (no design tokens, no grid, no defined scale), recommend establishing the system before continuing to build components.
- Never approve a mobile screen that shows a blank state while content loads — always require a skeleton or spinner.
- Never approve a feed screen that does not restore scroll position on return.
- Never approve a composition screen (post/comment/message) that does not implement auto-save.
- Never approve interactive `<div>` elements without semantic role, tabIndex, and keyboard handler — accessibility is a requirement, not a nice-to-have.
