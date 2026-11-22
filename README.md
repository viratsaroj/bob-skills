# Bob Skills Collection

A complete collection of **63 skills** for [IBM Bob](https://www.ibm.com/products/bob) (and compatible agentic AI tools). Skills are structured instruction packs that activate automatically when you trigger the right context — giving the AI deep, task-specific knowledge without you having to explain everything.

> Built and maintained by [@viratsaroj](https://github.com/viratsaroj)

---

## What is a Skill?

A **skill** is a folder containing a `SKILL.md` file (and optional reference docs, templates, and scripts). When Bob detects a matching intent in your message, it loads the skill and gains expert-level, up-to-date instructions for that task — going far beyond its base training.

**Compatible with:** IBM Bob · Any agent that supports `~/.claude/skills/`, `~/.agents/skills/`, or `~/.bob/skills/` skill loading.

---

## Quick Install

### Option 1 — Clone directly into Bob's skills folder

```bash
# For IBM Bob / Claude Code (primary location)
git clone https://github.com/viratsaroj/bob-skills.git
cp -r bob-skills/skills/* ~/.claude/skills/

# For IBM Agents (secondary location)
cp -r bob-skills/skills/* ~/.agents/skills/

# For Bob (another location Bob reads)
cp -r bob-skills/skills/* ~/.bob/skills/
```

### Option 2 — Clone anywhere and symlink

```bash
git clone https://github.com/viratsaroj/bob-skills.git ~/bob-skills

# Symlink each skill you want
ln -s ~/bob-skills/skills/founder-board ~/.claude/skills/founder-board
ln -s ~/bob-skills/skills/agency-seo ~/.claude/skills/agency-seo
# ... etc
```

### Option 3 — Pick only what you need

Download individual skill folders from the `skills/` directory and place them in `~/.claude/skills/` on your machine.

---

## Skill Catalogue

### 🏢 Agency Skills (26 skills)

A complete agency operating system — from client proposals to delivery:

| Skill | What it does |
|-------|-------------|
| `agency-accessibility` | Audit and fix accessibility (WCAG) in web products |
| `agency-admin-panel` | Design and build internal admin dashboards |
| `agency-ai-discoverability` | Optimise products for AI search and discoverability |
| `agency-api-design` | Design clean, consistent REST and GraphQL APIs |
| `agency-client-proposals` | Write winning client proposals and SOWs |
| `agency-code-review` | Structured, actionable code review process |
| `agency-community-launch` | Launch and grow a community around a product |
| `agency-content-strategy` | Plan content that drives traffic and conversions |
| `agency-copywriting` | Write high-converting copy for web, ads, emails |
| `agency-cybersecurity` | Security audit, threat model, hardening recommendations |
| `agency-database-design` | Schema design, indexing, query optimisation |
| `agency-devops` | CI/CD pipelines, infrastructure, deployment automation |
| `agency-digital-marketing` | Paid and organic digital marketing strategy |
| `agency-directory-listings` | Get listed in directories and aggregators |
| `agency-legal` | Contracts, terms of service, privacy policies, compliance |
| `agency-performance` | Web performance audit and optimisation (Core Web Vitals) |
| `agency-pr-and-mentions` | PR strategy, press outreach, earned media |
| `agency-project-scoping` | Scope projects, estimate effort, define deliverables |
| `agency-seo` | Technical SEO, on-page, link building strategy |
| `agency-social-media` | Social media strategy, content calendars, growth |
| `agency-sprint-planning` | Agile sprint planning, backlog grooming, velocity |
| `agency-technical-writing` | Docs, READMEs, API references, guides |
| `agency-testing-qa` | Test plans, QA processes, automated testing strategy |
| `agency-ui-design-review` | UI design critique and improvement recommendations |
| `agency-ux-research` | User research, interviews, usability testing |
| `agency-virtual-company` | Run a virtual agency/company — ops, hiring, culture |

### 🤖 AI Meta-Skills

| Skill | What it does |
|-------|-------------|
| `llm-council` | Convene a council of LLM personas to debate a question |
| `unslop` | Rewrite AI-generated slop into sharp, human-sounding prose |

### 🏗️ AWS Infrastructure

| Skill | What it does |
|-------|-------------|
| `aws-cdk` | Author, deploy, and troubleshoot AWS CDK stacks (TypeScript/Python) |
| `aws-cloudformation` | Write, validate, and troubleshoot CloudFormation templates |
| `aws-iam` | Write IAM policies, roles, trust conditions — corrects common AI mistakes |
| `aws-compute` | EC2 instance selection, Auto Scaling, AMIs, Systems Manager |
| `aws-containers` | EKS, ECS, Fargate, ECR, Elastic Beanstalk — full container lifecycle |
| `aws-serverless` | Lambda, API Gateway, EventBridge, Step Functions, event-driven patterns |
| `aws-networking` | Route 53, CloudFront, Transit Gateway, WAF, Shield, Direct Connect |
| `aws-storage` | S3, EFS, EBS, FSx — select, configure, optimise, troubleshoot |
| `aws-database` | Aurora, DynamoDB, RDS, Neptune, ElastiCache — choose and operate |

### 🤖 AWS AI & ML

| Skill | What it does |
|-------|-------------|
| `amazon-bedrock` | Converse API, Knowledge Bases, Agents, Guardrails, AgentCore, payments |
| `aws-ai-ml` | SageMaker fine-tuning (SFT/DPO/RLVR), model deployment, evaluation |

### 🔐 AWS Security & Auth

| Skill | What it does |
|-------|-------------|
| `aws-auth` | Cognito user/identity pools, OAuth 2.0, MFA, social/SAML federation |
| `aws-security` | Security Hub, GuardDuty, Inspector, Macie, Detective, Security Lake |

### 📊 AWS Observability & DevOps

| Skill | What it does |
|-------|-------------|
| `aws-observability` | CloudWatch Omni, Application Signals, Dynamic Instrumentation, alarms |
| `setting-up-cloudwatch-observability` | First-time Omni Space setup, ADOT instrumentation, Slack/MCP integration |
| `aws-deployment` | CodePipeline, CodeBuild, CodeDeploy, CodeArtifact, CodeConnections |

### 💰 AWS Cost & Messaging

| Skill | What it does |
|-------|-------------|
| `aws-billing-and-cost-management` | Cost Explorer, Savings Plans, Compute Optimizer, budgets, CUR/Athena |
| `aws-messaging-and-streaming` | SQS, SNS, EventBridge, Kinesis, MSK, MQ |

### 🚀 AWS SDK & Deployment

| Skill | What it does |
|-------|-------------|
| `aws-sdk-python-usage` | boto3/botocore patterns — clients, paginators, waiters, S3 transfers |
| `aws-sdk-js-v3-usage` | @aws-sdk/* JavaScript/TypeScript v3 patterns |
| `aws-sdk-swift-usage` | aws-sdk-swift patterns |
| `aws-blocks` | AWS Blocks Infrastructure-from-Code framework |
| `launch-with-aws` | Migrate vibe-coded / frontend apps to AWS (Lovable, Bolt, Replit, React) |
| `signing-in-to-aws` | `aws login`, credential setup, session expired fixes |

### 🧠 Founder / Business Skills (11 skills)

A complete **Founder Pack** — run each skill in sequence on a new idea, or individually:

| Skill | What it does |
|-------|-------------|
| `founder-board` | 3-member board (Offers, Monopoly, Product lenses) scores your idea and votes |
| `founder-competitors` | Competitive landscape analysis — who is already there, what moats exist |
| `founder-consumer` | 100 simulated buyer interviews — will they pay, why, why not |
| `founder-pricing` | Price architecture — anchoring, tiers, willingness-to-pay |
| `founder-offer` | Craft an irresistible offer (value stack, guarantee, urgency) |
| `founder-cfo` | Unit economics, runway, break-even, funding needs |
| `founder-marketing` | Go-to-market channels, CAC, messaging, launch sequence |
| `founder-brand` | Brand positioning, name, voice, visual direction |
| `founder-ops` | Operational model, hiring plan, tooling |
| `founder-launch` | Launch checklist, pre-launch waitlist, Day 1 execution |
| `founder-plan` | Wraps everything into a one-page operating plan |

**Suggested sequence:**
```
board → competitors → consumer → pricing → offer → cfo → marketing → brand → ops → launch → plan
```

---

## How Skills Are Triggered

Skills activate when Bob detects relevant intent in your message. Each `SKILL.md` has a `description:` field that lists trigger phrases. Examples:

- Say **"test my business idea"** → `founder-board` activates
- Say **"write an SEO strategy"** → `agency-seo` activates
- Say **"rewrite this to sound less like AI"** → `unslop` activates
- Say **"fine-tune a model on SageMaker"** → `aws-ai-ml` activates
- Say **"write a CDK stack for..."** → `aws-cdk` activates
- Say **"migrate my Lovable app to AWS"** → `launch-with-aws` activates

---

## Folder Structure

```
bob-skills/
├── README.md
└── skills/
    ├── agency-seo/
    │   └── SKILL.md          ← main skill instructions
    ├── amazon-bedrock/
    │   ├── SKILL.md
    │   ├── references/       ← deep-dive reference docs
    │   ├── assets/           ← templates, config files
    │   └── scripts/          ← helper scripts the skill may run
    ├── founder-board/
    │   └── SKILL.md
    └── ... (63 skills total)
```

---

## Keeping Skills Updated

```bash
cd bob-skills
git pull
cp -r skills/* ~/.claude/skills/
cp -r skills/* ~/.agents/skills/
cp -r skills/* ~/.bob/skills/
```

---

## Adding a New Skill

1. Create a folder under `skills/<your-skill-name>/`
2. Add a `SKILL.md` with this frontmatter:

```yaml
---
name: your-skill-name
description: >-
  One paragraph describing what this skill does and what phrases trigger it.
argument-hint: "<what to pass after the slash command, if any>"
---

# Your Skill Title

Your instructions here...
```

3. Optionally add `references/`, `assets/`, or `scripts/` subfolders.
4. Place the folder in `~/.claude/skills/` and restart Bob.

---

## License

MIT — use freely, modify, contribute back.

---

## Contributing

PRs welcome. If you've built a skill that works well for a specific domain, open a PR and add it to the catalogue. Keep `SKILL.md` self-contained — the AI reads it verbatim, so clear prose beats clever abstraction.
