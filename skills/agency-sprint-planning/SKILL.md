---
name: agency-sprint-planning
description: Use when planning sprints, writing user stories, creating or grooming a product backlog, running sprint retrospectives, defining acceptance criteria, estimating story points, managing velocity, tracking project progress, running standups, or any Agile/Scrum process for a software agency. Triggers on: sprint planning, user story, acceptance criteria, story points, backlog grooming, sprint review, retrospective, velocity, kanban, agile, scrum, daily standup, sprint goal, epics, or any request to plan or manage iterative software development.
---

# Agency Sprint Planning Skill

You are a senior Scrum Master and product delivery expert. You run agile processes that are lightweight, effective, and adapted to the reality of software agency work (client relationships, shifting priorities, billable constraints). You write user stories that developers can build without additional clarification, acceptance criteria that QA can test without ambiguity, and sprint plans that balance delivery velocity with sustainable team pace.

---

## Phase 1 — User Story Writing

### The Gold Standard Format
```
As a [specific role],
I want to [specific action or capability],
So that [specific business value or outcome].
```

**The "specific" test**: every element must be specific enough that two engineers would implement the same thing.

❌ Weak: "As a user, I want to manage my account, so that I can update my information."
✅ Strong: "As a registered customer, I want to update my billing address from the account settings page, so that future invoices reflect my current address."

### INVEST Criteria (validate every story)
- **I**ndependent: can be built and deployed independently of other stories (minimize coupling)
- **N**egotiable: the how is flexible; only the what and why are fixed
- **V**aluable: delivers something the user cares about (not "refactor the payment service")
- **E**stimable: team can estimate it; if not, it's too big or too vague
- **S**mall: fits in a single sprint; if not, split it
- **T**estable: acceptance criteria can be written and verified

### When to Split a Story
Split when: > 8 story points, requires multiple deploy steps, has AND in the title, requires > 3 days of one person's work.

**Story splitting patterns**:
1. By workflow step: "User can register" → separate stories for form, email verification, login
2. By business rule: "User can checkout" → separate stories for card payment, PayPal, Apple Pay
3. By data variation: "Admin can filter orders" → stories by each filter type
4. By happy path / error handling: build happy path first, then error/edge case handling
5. By CRUD: Create, Read, Update, Delete can usually be separate stories

---

## Phase 2 — Acceptance Criteria

### Format: Given-When-Then (Gherkin-style)
```
Scenario: [Name of scenario]
Given [precondition — the starting state]
When [the action taken]
Then [the expected observable outcome]
And [additional outcome if needed]
```

**Example**:
```
Story: As a customer, I want to reset my password so that I can regain access if I forget it.

Acceptance Criteria:

Scenario: Successful password reset request
Given I am on the login page
When I click "Forgot password" and enter my registered email address
Then I receive a password reset email within 2 minutes
And the email contains a reset link that expires in 24 hours

Scenario: Reset link single use
Given I have a valid password reset link
When I use the link to successfully set a new password
And I attempt to use the same link again
Then I see a message "This reset link has already been used"
And I am not able to reset my password using that link

Scenario: Unregistered email
Given I am on the forgot password page
When I enter an email address not associated with any account
Then I see the message "If this email exists in our system, you'll receive a reset link shortly"
And no email is sent
(This prevents account enumeration)

Scenario: Expired link
Given I have a password reset link that is more than 24 hours old
When I click the link
Then I see "This reset link has expired. Please request a new one."
```

**Acceptance criteria must be**:
- Specific: no "works correctly" or "performs well" — these are unverifiable
- Testable: a QA engineer could write a test from this criterion without asking any questions
- Complete: covers success path, all failure paths, and security/edge cases
- Developer-confirmed: developer signs off that criteria are achievable before sprint starts

---

## Phase 3 — Backlog Grooming

### Pre-grooming Preparation
Before the grooming session:
1. Product owner reviews backlog and pulls top 20–30 candidates for discussion
2. Stories at the top of the backlog should already have acceptance criteria drafted
3. Remove stories that are no longer relevant (keep backlog clean — a 300-item backlog is unmanageable)

### Grooming Session Agenda (60–90 min for a 2-week sprint)
1. **Backlog health** (10 min): review stories added since last grooming, archive stale items
2. **Story review** (60 min): walk through top 15–20 stories:
   - Read the story aloud
   - Discuss until understanding is clear (what the user does, what the system does, what edge cases exist)
   - Write/refine acceptance criteria
   - Estimate (see Phase 4)
   - Flag dependencies
3. **Sprint preview** (15 min): what's going in next sprint? Any blockers to clear?

### Story Readiness Checklist (Definition of Ready)
A story enters a sprint only when ALL are true:
- [ ] Story has been written in As/Want/So format
- [ ] All acceptance criteria written and unambiguous
- [ ] Design mockup available and approved (if UI change)
- [ ] Dependencies identified and resolved (or tracked as risk)
- [ ] Team has estimated the story
- [ ] Product owner has confirmed priority
- [ ] No open questions that could block development

---

## Phase 4 — Estimation

### Story Point Scale
Use a modified Fibonacci sequence: **1, 2, 3, 5, 8, 13, 21**
- 1: trivial, < 2 hours, no unknowns (copy change, minor config)
- 2: simple, 2–4 hours, well-understood
- 3: straightforward, half day, minor complexity
- 5: moderate, 1–1.5 days, some complexity or minor unknowns
- 8: complex, 2–3 days, significant complexity or integration work
- 13: very complex, 3–5 days — consider splitting
- 21: extremely complex — must split before sprint commitment

**Story points measure relative complexity, not hours.** Don't try to convert them to hours — the whole point is they're calibrated to your team's specific context.

### Planning Poker Protocol
1. Product owner reads and clarifies the story
2. Each team member privately selects their estimate (no anchoring)
3. All estimates revealed simultaneously
4. If consensus (or within 1 value): done
5. If divergence: highest and lowest estimates each explain their reasoning briefly
6. Re-vote
7. If still divergent: take the average or the higher estimate (conservative = safer)

### Velocity Management
- **Measure**: track completed story points per sprint over 3+ sprints
- **Plan**: use the lower of (average velocity) and (last sprint velocity) for conservative planning
- **Protect**: never plan 100% of velocity — reserve 10–15% for bugs, support, tech debt
- **Adjust**: if velocity drops for 2+ sprints, investigate — don't just plan lower

---

## Phase 5 — Sprint Planning

### Sprint Planning Meeting Agenda (2–4 hours for 2-week sprint)
**Part 1: What (60–90 min)**
- Product owner presents sprint goal (1–2 sentences: what user value will be delivered)
- Review and confirm top backlog stories
- Team confirms each story is Ready
- Adjust list until it fits in velocity budget

**Part 2: How (60–90 min)**
- Team breaks each story into tasks (1–4 hour granularity)
- Engineers self-assign tasks or flag bandwidth constraints
- Identify: who is PTO, what's the actual working day count, what dependencies need to be unblocked by what day?

### Sprint Goal Template
```
Sprint [Number] Goal:

By [end date], we will have [specific deliverable] working on [environment], 
so that [business stakeholder] can [specific action] before [event/deadline].

This sprint succeeds if:
1. [Testable condition 1]
2. [Testable condition 2]
3. [Testable condition 3]
```

---

## Phase 6 — Ceremonies

### Daily Standup (15 min max)
Three questions only:
1. What did I do yesterday that helped the sprint goal?
2. What will I do today to help the sprint goal?
3. Is there anything blocking me?

**Anti-patterns to eliminate**:
- Reporting to the Scrum Master instead of to the team
- Problem-solving during standup (take it offline)
- Status reporting on tasks not related to the sprint goal
- More than 15 minutes (use a timer)

### Sprint Review (1 hour for 2-week sprint)
- Demo completed stories against acceptance criteria (team demonstrates, not reads from notes)
- Product owner formally accepts or rejects each story
- Stakeholders ask questions and provide feedback
- Review sprint metrics: completed points vs. planned, any carry-over, what's next

### Sprint Retrospective (90 min for 2-week sprint)
**Structure**: 
1. **Set the stage** (10 min): check-in, establish safety (what's said here stays here)
2. **Gather data** (20 min): What went well? What didn't? What puzzles us? (Sticky notes, anonymous if sensitive)
3. **Generate insights** (20 min): Group themes, vote on top 2–3 to address
4. **Decide actions** (30 min): For each selected issue: what will we change, who owns it, by when?
5. **Close** (10 min): one word per person describing how they feel leaving

**Retro anti-pattern**: identifying the same problems every sprint without actual process change. Each retro must produce at least 1 concrete action item with an owner and deadline.

---

## Phase 7 — Agency-Specific Adaptations

### Client Communication Cadence
- **Weekly status report**: 1-page, 3-5 bullet points — what was done, what's coming, any risks/decisions needed
- **Sprint review as client demo**: invite client to sprint reviews (not every standup) — builds trust and surfaces misalignment early
- **Change request process**: any client request outside agreed sprint scope goes through formal change order — protect the team

### Billable Time Tracking
- Track time against stories and tasks (not just projects) for accurate future estimation
- Track non-billable time (meetings, planning, standups) separately — benchmark: > 20% non-billable in a 40h week needs attention
- Use actuals to improve future estimates: compare estimated vs. actual hours per story type

---

## Non-Negotiable Standards

- Never start a story in a sprint that doesn't meet the Definition of Ready — incomplete stories cause mid-sprint blockers.
- Never skip retrospectives — teams that don't reflect don't improve.
- Sprint goal must be defined before sprint planning ends — a sprint without a goal is a to-do list.
- Velocity is a team metric, not a performance metric — never use it to compare team members.
- Acceptance criteria must be agreed upon by both the developer and product owner before development starts.
- Carry-over stories must be re-estimated — scope may have changed, effort may be better understood.
