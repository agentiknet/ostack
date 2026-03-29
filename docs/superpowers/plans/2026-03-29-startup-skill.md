# /startup Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `/startup` ostack skill — a single invocation that takes the user from an empty folder to a fully provisioned, deployed, and documented project.

**Architecture:** Router skill + route modules. One `SKILL.md.tmpl` orchestrates the flow. Route-specific architecture topics, provisioning steps, and generators live in separate `.md` prompt fragments read by the main skill at runtime via the `Read` tool. Only the main `SKILL.md.tmpl` goes through the `gen-skill-docs` pipeline.

**Tech Stack:** ostack skill template system (YAML frontmatter + `{{PREAMBLE}}` / `{{SLUG_EVAL}}` placeholders), `gcloud` CLI, `gh` CLI, PostHog MCP tools, Stripe MCP tools, Google Analytics Admin API, Search Console API, Resend API.

**Spec:** `docs/superpowers/specs/2026-03-29-startup-skill-design.md`

---

## File Structure

All files live under `startup/` in the ostack repo root:

| File | Responsibility |
|------|---------------|
| `startup/SKILL.md.tmpl` | Main skill template — frontmatter, interview phase, architecture router, provisioning orchestrator, file generation, summary. Goes through `gen-skill-docs` pipeline. |
| `startup/routes/saas.md` | SaaS architecture topics: multi-tenancy, billing model, roles, onboarding, GDPR |
| `startup/routes/landing.md` | Landing page topics: static vs SSR, CMS, forms, SEO, performance budget |
| `startup/routes/ai.md` | AI/ML topics: GPU strategy, dataset versioning, experiment tracking, model serving, licensing |
| `startup/routes/internal-tool.md` | Internal tool topics: access control (IAP/SSO), build vs buy, data sources |
| `startup/routes/api.md` | API service topics: OpenAPI, rate limiting, API key mgmt, versioning, SDK gen |
| `startup/routes/cli.md` | CLI tool topics: distribution, language, config format, update mechanism, completions |
| `startup/provisioning/gcp-project.md` | GCP project creation, billing link, API enabling, service account |
| `startup/provisioning/github.md` | Git init + GitHub repo creation via `gh` |
| `startup/provisioning/dns.md` | Cloud DNS zone creation, nameserver extraction |
| `startup/provisioning/cloud-run.md` | Cloud Run deploy + custom domain mapping + SSL |
| `startup/provisioning/analytics.md` | GA4 property + data stream + Search Console verification |
| `startup/provisioning/posthog.md` | PostHog org + project creation via MCP |
| `startup/provisioning/stripe.md` | Stripe product + price + webhook + portal via MCP |
| `startup/provisioning/email.md` | Resend API key + DNS records + welcome template |
| `startup/provisioning/monitoring.md` | GCP Uptime Check + Cloud Monitoring alerts |
| `startup/provisioning/cicd.md` | CI/CD pipeline setup (GitHub Actions or Cloud Build) |
| `startup/generators/readme.md` | README generation rules: ASCII art, best practices, structure |
| `startup/generators/project-json.md` | project.json schema + PROJECT.md generation rules |
| `startup/generators/claude-md.md` | CLAUDE.md seeding rules |
| `startup/generators/adr.md` | Architecture Decision Record format |
| `startup/generators/gitignore.md` | Stack-tailored .gitignore rules per language/framework |
| `startup/generators/scaffold.md` | Working skeleton generation rules per route |
| `startup/discovery/service-suggestions.md` | Smart "did you forget about..." detection rules |

---

## Task Dependency Graph

```
Task 1 (directory skeleton)
  → Task 2 (main SKILL.md.tmpl — Phase 1: Interview)
  → Task 3 (route modules — 6 files)
  → Task 4 (main SKILL.md.tmpl — Phase 2: Architecture conversation)
  → Task 5 (service discovery module)
  → Task 6 (provisioning modules — GCP, GitHub, DNS)
  → Task 7 (provisioning modules — Cloud Run, Analytics, PostHog)
  → Task 8 (provisioning modules — Stripe, Email, Monitoring, CI/CD)
  → Task 9 (generator modules — readme, project-json, claude-md, adr, gitignore, scaffold)
  → Task 10 (main SKILL.md.tmpl — Phase 3-4: File generation, ship, summary)
  → Task 11 (gen-skill-docs + validation)
  → Task 12 (root SKILL.md.tmpl update + CLAUDE.md update)
```

Tasks 3, 5, 6, 7, 8, 9 can be parallelized since they're independent files. But the main SKILL.md.tmpl (Tasks 2, 4, 10) must be built sequentially since it references all modules.

---

### Task 1: Create Directory Skeleton

**Files:**
- Create: `startup/` and all subdirectories

- [ ] **Step 1: Create all directories**

```bash
mkdir -p startup/routes startup/provisioning startup/generators startup/discovery
```

- [ ] **Step 2: Verify structure**

```bash
find startup -type d | sort
```

Expected output:
```
startup
startup/discovery
startup/generators
startup/provisioning
startup/routes
```

- [ ] **Step 3: Commit**

```bash
git add startup/
git commit -m "chore: create startup skill directory skeleton"
```

---

### Task 2: Main SKILL.md.tmpl — Phase 1 (Interview)

**Files:**
- Create: `startup/SKILL.md.tmpl`

This task creates the frontmatter and the entire Phase 1 interview flow. The file will be extended in Tasks 4 and 10.

- [ ] **Step 1: Write the frontmatter and Phase 1 interview**

Create `startup/SKILL.md.tmpl` with this content:

```markdown
---
name: startup
version: 1.0.0
description: |
  Project bootstrap — from empty folder to fully provisioned infrastructure.
  Six project archetypes: SaaS, landing page, AI/ML, internal tool, API service,
  CLI tool. Creates GCP project, service account, Cloud Run, Cloud DNS, GA4,
  Search Console, PostHog, optional Stripe and Resend. Generates project.json,
  CLAUDE.md, README with ASCII art, ADRs, and working code scaffold.
  Use when: "new project", "start a project", "bootstrap", "startup", "init project",
  "set up a new app", "create a new service".
  Proactively suggest when the user is in an empty directory or mentions starting
  something new.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
---

{{PREAMBLE}}

# /startup — From Empty Folder to Running Project

You are a **senior software architect and DevOps engineer**. Your job is to take the user from an empty folder to a fully provisioned, deployed, and documented project in a single session. You are opinionated, recommend durable choices, and explain trade-offs clearly.

**This is NOT a brainstorming tool.** For idea validation, use `/office-hours`. This skill is operational: collect facts, make architecture decisions, provision infrastructure, generate code, push to GitHub.

---

## Phase 0: Pre-checks

Check if this is truly a new project or a re-run:

```bash
ls project.json 2>/dev/null && echo "PROJECT_EXISTS" || echo "NEW_PROJECT"
ls CLAUDE.md 2>/dev/null && echo "CLAUDE_EXISTS" || echo "NO_CLAUDE"
ls .git 2>/dev/null && echo "GIT_EXISTS" || echo "NO_GIT"
```

**If project.json exists:** This is a re-run. Read `project.json`, show what's already set up, identify what's missing, and offer to fill gaps. Skip the interview for any field that already has a value.

**If no project.json:** This is a fresh start. Proceed with the full interview.

---

## Phase 1: Project Interview

Ask these questions via AskUserQuestion, **one at a time**. Do not batch them. Wait for each answer before asking the next.

Store all answers mentally — they will be used to drive every subsequent phase.

### Question 1: Project Name

> What's the name of your project? (Human-readable, e.g., "My SaaS App")

### Question 2: Project Slug

Auto-suggest a slug from the name (lowercase, hyphens, no spaces). Example: "My SaaS App" → `my-saas-app`.

> Suggested slug: `{auto-slug}`. This will be used for the GCP project ID, Cloud Run service name, GitHub repo name, and folder name. Want to use this or override it?

The slug must be:
- Lowercase letters, numbers, and hyphens only
- 6-30 characters (GCP project ID constraint)
- Globally unique on GCP (if the user picks one that's taken, they'll find out in the provisioning step and can retry)

### Question 3: Description

> One-line description: what does this project do?

### Question 4: Project Type

> What type of project is this?
>
> - **A)** SaaS web app (auth, dashboard, billing)
> - **B)** Landing page / marketing site
> - **C)** AI / model training
> - **D)** Internal tool
> - **E)** API service / backend
> - **F)** CLI tool

Store the answer as one of: `saas`, `landing`, `ai`, `internal-tool`, `api`, `cli`.

### Question 5: Monetization

> Is this a paid product or free?
>
> - **A)** Paid — users will pay (activates Stripe setup)
> - **B)** Free — no payments
> - **C)** Not sure yet — skip Stripe for now, can add later

If A: Stripe provisioning will run later.

### Question 6: Email

> Does this project need to send emails? (Welcome emails, notifications, transactional)
>
> - **A)** Yes (activates Resend setup)
> - **B)** No
> - **C)** Not sure yet — skip for now

If A: Resend provisioning will run later.

### Question 7: Geography

> Where is your target audience primarily located?
>
> - **A)** Europe
> - **B)** North America
> - **C)** Asia-Pacific
> - **D)** Global / mixed
> - **E)** Doesn't matter (internal tool, personal project)

Use this to recommend a GCP region.

### Question 8: GCP Region

Based on geography answer, recommend a region:
- Europe → `europe-west1` (Belgium) or `europe-west9` (Paris)
- North America → `us-central1` (Iowa) or `us-east1` (South Carolina)
- Asia-Pacific → `asia-east1` (Taiwan) or `asia-southeast1` (Singapore)
- Global → `us-central1` (lowest latency on average)
- Doesn't matter → `europe-west1`

> Recommended GCP region: `{region}` ({reason}). Use this or pick another?

### Question 9: Domain

> Do you have a domain name for this project?
>
> - **A)** Yes — already bought on Namecheap: {domain}
> - **B)** Not yet — will buy later
> - **C)** No domain needed

If A: Cloud DNS zone will be created, nameservers will be output for Namecheap.
If B or C: Skip DNS provisioning, use Cloud Run default URL.

### Question 10: GitHub

> GitHub repo setup:
>
> 1. Public or private?
> 2. Which organization? (or personal account)

### Question 11: Environments

> What environments do you need?
>
> - **A)** Just production (ship fast)
> - **B)** Production + staging
> - **C)** Production + staging + dev

### Question 12: Legal

> Does this project need legal pages? (Privacy policy, terms of service — required if tracking users with GA4/PostHog)
>
> - **A)** Yes — generate placeholder pages with TODOs
> - **B)** No — I'll handle legal separately
> - **C)** Not sure — generate placeholders to be safe

---

After all questions are answered, summarize:

> **Project summary:**
> - Name: {name}
> - Slug: {slug}
> - Type: {type}
> - Description: {description}
> - GCP region: {region}
> - Domain: {domain or "none"}
> - GitHub: {org}/{slug} ({visibility})
> - Environments: {environments}
> - Stripe: {yes/no}
> - Resend: {yes/no}
> - Legal pages: {yes/no}
>
> **Moving to architecture conversation. After that, everything provisions automatically.**

Proceed to Phase 2.
```

- [ ] **Step 2: Verify the file parses correctly**

The file should have valid YAML frontmatter and use only known template placeholders (`{{PREAMBLE}}`).

```bash
head -25 startup/SKILL.md.tmpl
```

- [ ] **Step 3: Commit**

```bash
git add startup/SKILL.md.tmpl
git commit -m "feat(startup): add SKILL.md.tmpl with frontmatter and Phase 1 interview"
```

---

### Task 3: Route Modules (6 files)

**Files:**
- Create: `startup/routes/saas.md`
- Create: `startup/routes/landing.md`
- Create: `startup/routes/ai.md`
- Create: `startup/routes/internal-tool.md`
- Create: `startup/routes/api.md`
- Create: `startup/routes/cli.md`

Each route module is a prompt fragment that the main skill reads at runtime. It defines:
1. The architecture topics specific to this project type
2. Default recommendations for each topic
3. The scaffold specification for this route

These are NOT templates — they don't go through gen-skill-docs. They're plain markdown read via the `Read` tool.

- [ ] **Step 1: Write `startup/routes/saas.md`**

```markdown
# SaaS Web App — Architecture Topics

Read this file when the user selected project type: **SaaS web app**.

Walk through each topic below **one at a time** via AskUserQuestion. For each:
- State your recommendation and why
- Present 1-2 alternatives with trade-offs
- Wait for sign-off before moving to the next topic

Record every decision — these become the ADR in Phase 3.

---

## Common Topics (ask these first)

### 1. Language & Framework

**Recommendation:** TypeScript + Next.js (App Router, SSR-first)

**Why:** User's default for web projects. Largest ecosystem, best SSR support, deploys cleanly to Cloud Run via Docker. App Router is the current standard.

**Alternatives:**
- SvelteKit — lighter, faster DX, smaller community
- Remix — good SSR, less ecosystem than Next.js

### 2. Database

**Recommendation:** Depends on scale expectations.

Ask the user:
> How much data and how many users do you expect in the first 6 months?
>
> - **A)** Small (< 1000 users, simple data) → **Firestore** (serverless, zero ops, generous free tier)
> - **B)** Medium (1K-100K users, relational data) → **Cloud SQL Postgres** (reliable, relational, good tooling)
> - **C)** Large / complex queries → **Cloud SQL Postgres** or **AlloyDB** (Postgres-compatible, better performance)

**ORM explanation for the user:**
An ORM (Object-Relational Mapper) is a library in your code that lets you interact with your database using your programming language instead of raw SQL. Think of it as a translator between your code and your database.

For TypeScript:
- **Drizzle** (recommended) — lightweight, fast, close to SQL, great TypeScript types
- **Prisma** — most popular, auto-generates types, slightly heavier

### 3. Authentication

**Recommendation:** Depends on complexity.

Ask:
> What kind of authentication do you need?
>
> - **A)** Simple (email/password + Google login) → **Firebase Auth** (GCP-native, free tier, easy)
> - **B)** Advanced (SSO, SAML, MFA, org management) → **Clerk** (hosted, excellent DX, paid)
> - **C)** Full control (custom auth logic) → **Auth.js / NextAuth** (open source, self-hosted)

### 4. Hosting & Scaling

**Default:** Google Cloud Run.

Flag if NOT adapted: If the project needs WebSocket connections, long-running background jobs (> 60 min), or GPU inference, Cloud Run may not be ideal. In those cases, discuss:
- GKE for complex orchestration
- Compute Engine for GPU/long-running
- Cloud Functions for event-driven micro-tasks

For standard SaaS: Cloud Run is perfect. Containers, auto-scaling, pay-per-use, generous free tier.

### 5. CI/CD

Ask:
> How should builds and deploys work?
>
> - **A)** GitHub Actions (push to main → build → deploy to Cloud Run) — most flexible, familiar
> - **B)** Cloud Build triggers (GCP-native, triggered by GitHub push) — tighter GCP integration
>
> **Recommendation:** GitHub Actions. More flexibility, better ecosystem, easier to debug. Cloud Build is fine too but fewer community actions.

---

## SaaS-Specific Topics (ask after common topics)

### 6. Multi-Tenancy Model

Ask:
> How should tenant data be isolated?
>
> - **A)** Shared database, tenant column (simplest, recommended for most SaaS)
> - **B)** Schema-per-tenant (stronger isolation, more complexity)
> - **C)** Database-per-tenant (maximum isolation, highest cost/complexity)
>
> **Recommendation:** A (shared DB with tenant column). Covers 95% of SaaS use cases. Easy to query, easy to manage. Move to B or C only if you have enterprise clients with strict data isolation requirements.

### 7. Subscription & Billing Model

Ask:
> How will users pay?
>
> - **A)** Monthly/annual subscription — flat price per plan
> - **B)** Seat-based — price per user per month
> - **C)** Usage-based — pay for what you use (API calls, storage, etc.)
> - **D)** Freemium — free tier + paid upgrade
> - **E)** Trial → paid — free trial, then subscription
>
> This determines how Stripe products/prices are configured.

Also ask:
> How many pricing tiers? (e.g., Free / Pro / Enterprise, or just one plan)

### 8. User Roles & Permissions

Ask:
> What user roles does your app need?
>
> - **A)** Simple — just users (no roles)
> - **B)** Basic — admin + member
> - **C)** Advanced — admin + member + viewer + custom roles
>
> **Recommendation:** Start with B. You can always add more roles later. Over-engineering roles early is a common SaaS trap.

### 9. Onboarding Flow

Ask:
> How should new users get started after signup?
>
> - **A)** Minimal — land on dashboard, explore freely
> - **B)** Guided — step-by-step wizard (e.g., "Set up your workspace" → "Invite team" → "Create first project")
> - **C)** Template-based — pick a template/preset to get started with pre-filled data
>
> **Recommendation:** B for most SaaS. A good onboarding flow dramatically improves activation rates.

### 10. GDPR & Data Export

Ask:
> Do you need GDPR compliance features?
>
> - **A)** Yes — data export, account deletion, consent management
> - **B)** Not yet — but plan for it (add the database columns, skip the UI)
> - **C)** No — not targeting EU users
>
> **Recommendation:** B. The database schema cost is near-zero, and retrofitting GDPR compliance later is painful.

---

## Scaffold Spec

After architecture decisions are made, the SaaS scaffold should include:

```
{project-slug}/
├── src/app/                    # Next.js App Router
│   ├── layout.tsx              # Root layout with providers (PostHog, GA4)
│   ├── page.tsx                # Landing/home page
│   ├── api/
│   │   ├── health/route.ts     # Health check endpoint (GET /api/health)
│   │   └── webhooks/
│   │       └── stripe/route.ts # Stripe webhook handler (if paid)
│   ├── (auth)/                 # Auth routes (login, signup)
│   └── (dashboard)/            # Protected dashboard routes
├── src/lib/
│   ├── posthog.ts              # PostHog client init
│   ├── analytics.ts            # GA4 helpers
│   ├── stripe.ts               # Stripe client init (if paid)
│   └── email.ts                # Resend client init (if email)
├── Dockerfile                  # Multi-stage build for Cloud Run
├── .dockerignore
├── next.config.ts
├── package.json
├── tsconfig.json
├── biome.json                  # Linter/formatter config
└── [legal pages if requested]
```
```

- [ ] **Step 2: Write `startup/routes/landing.md`**

```markdown
# Landing Page — Architecture Topics

Read this file when the user selected project type: **Landing page / marketing site**.

Walk through each topic below **one at a time** via AskUserQuestion.

---

## Common Topics

### 1. Language & Framework

Ask:
> Does this landing page need dynamic features (auth, forms with server processing, personalization)?
>
> - **A)** No — pure static content → **Consider plain HTML/CSS or Astro** (fastest, cheapest, simplest)
> - **B)** Some dynamic features → **Next.js** (user's default, SSR, good SEO)
> - **C)** Heavily dynamic → **Next.js** (definitely)
>
> **Note:** If the answer is A, discuss whether Next.js is overkill. A static site generator (Astro) or even plain HTML can be faster, cheaper, and simpler to deploy. But if the user prefers Next.js for consistency, that's fine too.

### 2. Database

For landing pages, a database is often optional:
> Do you need to store data? (Waitlist signups, form submissions, content)
>
> - **A)** No data storage needed
> - **B)** Simple form data → **Firestore** (serverless, free tier, zero config)
> - **C)** Blog/content → depends on CMS choice (next topic)

### 3. Authentication

Usually not needed for landing pages:
> Does anyone need to log in to this site?
>
> - **A)** No — public site, no auth needed (most landing pages)
> - **B)** Yes — admin panel for content editing → lightweight auth

### 4. Hosting & Scaling

**Default:** Cloud Run.

**However:** For a purely static landing page, Firebase Hosting is simpler and free. Discuss:
- Static site → Firebase Hosting (free, global CDN, instant deploys)
- Dynamic/SSR → Cloud Run (as usual)

### 5. CI/CD

Same as common: GitHub Actions or Cloud Build. For static sites, consider:
- Direct `firebase deploy` from GitHub Actions (simpler than Cloud Run)

---

## Landing-Page-Specific Topics

### 6. CMS / Content Management

Ask:
> How will you manage the page content?
>
> - **A)** Hardcoded in code (simplest — content changes = code changes)
> - **B)** MDX files (Markdown + JSX — great for blogs, easy to edit)
> - **C)** Headless CMS (Contentful, Sanity, Notion as CMS) — non-technical editors can update
> - **D)** Not sure
>
> **Recommendation:** A for simple landing pages. B if you'll have a blog. C only if non-technical people need to edit content.

### 7. Form Handling

Ask:
> How should forms work? (Contact forms, waitlist signups, etc.)
>
> - **A)** No forms needed
> - **B)** Simple — collect email, store in Firestore
> - **C)** Advanced — multi-field forms, validation, file uploads
>
> If B: Scaffold a server action or API route that writes to Firestore.
> If C: Discuss form library (react-hook-form) and storage.

### 8. SEO Strategy

This is critical for landing pages. Configure from day one:
> What's the SEO priority?
>
> - **A)** High — this page needs to rank in search (most landing pages)
> - **B)** Low — this is behind auth or for a specific audience
>
> If A: Set up from day one:
> - Meta tags (title, description, OG tags) in layout
> - Sitemap generation
> - robots.txt
> - Structured data (JSON-LD) for the homepage
> - Canonical URLs

### 9. Performance Budget

> Target Core Web Vitals scores:
> - LCP (Largest Contentful Paint): < 2.5s
> - FID (First Input Delay): < 100ms
> - CLS (Cumulative Layout Shift): < 0.1
>
> These are Google's "good" thresholds. The scaffold will be optimized for these by default (proper image optimization, font loading, minimal JS).

---

## Scaffold Spec

```
{project-slug}/
├── src/app/
│   ├── layout.tsx              # Root layout with meta tags, GA4, PostHog
│   ├── page.tsx                # Main landing page
│   ├── api/health/route.ts     # Health check
│   ├── sitemap.ts              # Auto-generated sitemap
│   └── robots.ts               # robots.txt
├── src/components/             # Reusable UI components
├── public/                     # Static assets
├── Dockerfile
├── .dockerignore
├── next.config.ts
├── package.json
├── tsconfig.json
└── biome.json
```
```

- [ ] **Step 3: Write `startup/routes/ai.md`**

```markdown
# AI / Model Training — Architecture Topics

Read this file when the user selected project type: **AI / model training**.

Walk through each topic below **one at a time** via AskUserQuestion.

**Important context:** The user works with Rust, C, and Python for AI projects, adapting to context. Language choice is a real discussion here, not a default.

---

## Common Topics

### 1. Language & Framework

Ask:
> What's the primary work for this AI project?
>
> - **A)** Training a model (data processing + training loop) → **Python** (PyTorch/JAX ecosystem, no alternative)
> - **B)** High-performance inference server → **Rust** (with Python bindings for model loading) or **Python** (FastAPI/vLLM)
> - **C)** Data pipeline / preprocessing → **Python** (pandas/polars) or **Rust** (for performance-critical ETL)
> - **D)** Research / experimentation → **Python** (Jupyter + PyTorch)
> - **E)** Mix — training in Python, serving in Rust/C
>
> **Recommendation:** Python is unavoidable for training (the ML ecosystem is Python). For serving, Rust is worth considering if latency matters. For most projects, pure Python is fine.

If Python:
> Package manager?
> - **A)** uv (recommended — fast, modern, replaces pip+venv+poetry)
> - **B)** Poetry
> - **C)** pip + venv (simple, no extra tools)

### 2. Database

Ask:
> Does this project need a database? (Beyond file storage for datasets/models)
>
> - **A)** No — just file storage (models, datasets on GCS)
> - **B)** Metadata store — experiment results, run configs → **Firestore** or **SQLite**
> - **C)** Full application database — if this AI project includes a web frontend → choose as needed

### 3. Authentication

Usually minimal for AI projects:
> Who uses this?
>
> - **A)** Just me / my team (no auth needed, or basic API key)
> - **B)** External users via API (need API key management)
> - **C)** Web UI for users (need full auth — redirect to SaaS route topics)

### 4. Hosting & Scaling

**Cloud Run may NOT be the default here.** Discuss:
> What does deployment look like?
>
> - **A)** API server for inference (no GPU) → **Cloud Run** (fine, standard containers)
> - **B)** GPU inference → **Compute Engine with GPU** or **Vertex AI endpoints**
> - **C)** Training jobs → **Compute Engine with GPU** or **Vertex AI Training** or external (Lambda Labs, RunPod)
> - **D)** Batch processing → **Cloud Run Jobs** or **Compute Engine**
>
> **Flag:** If GPU is needed, discuss cost. Cloud GPUs are expensive. External providers (Lambda Labs, RunPod) can be 2-5x cheaper for training. GCP GPUs are better for inference (close to your other infra).

### 5. CI/CD

For AI projects, CI/CD is different:
> What needs to happen on push?
>
> - **A)** Standard: lint + test + deploy inference server → GitHub Actions
> - **B)** ML pipeline: lint + test + trigger training run → GitHub Actions + Vertex AI Pipelines
> - **C)** Minimal: just lint + test (manual deploys for now)

---

## AI-Specific Topics

### 6. GPU Strategy

Ask:
> Do you need GPUs? If yes, for what?
>
> - **A)** No GPUs — CPU-only inference or data processing
> - **B)** Training — need GPUs for model training
> - **C)** Inference — need GPUs for real-time inference
> - **D)** Both training and inference
>
> If B or D:
> - **Cloud GPUs (Vertex AI, Compute Engine):** Easy GCP integration, pay-per-minute, more expensive
> - **Lambda Labs:** Cheapest cloud GPUs, great for training, less integrated
> - **RunPod:** Serverless GPUs, good for burst training
> - **Local:** If you have a GPU machine, cheapest long-term
>
> **Recommendation:** Lambda Labs or RunPod for training (cost), GCP for inference (latency, integration).

### 7. Dataset Storage & Versioning

Ask:
> How large are your datasets and how do you want to manage them?
>
> - **A)** Small (< 10GB) → **GCS bucket** (simple, cheap)
> - **B)** Large (10GB-1TB) → **GCS bucket + DVC** (Data Version Control — git for data)
> - **C)** Very large (> 1TB) → **GCS bucket + manifest files** (track what's where without downloading everything)
> - **D)** Using HuggingFace datasets → **HuggingFace Hub** (built-in versioning)

### 8. Experiment Tracking

Ask:
> How do you want to track experiments (hyperparameters, metrics, model versions)?
>
> - **A)** Weights & Biases (W&B) — industry standard, free tier, great visualizations
> - **B)** MLflow — open source, self-hosted, more control
> - **C)** TensorBoard — basic, built into PyTorch/TF, no server needed
> - **D)** None — manual tracking (fine for small projects)
>
> **Recommendation:** W&B for serious projects (it's free for personal use). TensorBoard for quick experiments.

### 9. Model Serving & Inference

Ask:
> How will the trained model be used?
>
> - **A)** REST API endpoint → Cloud Run (CPU) or Vertex AI Endpoints (GPU)
> - **B)** Batch processing → Cloud Run Jobs or Vertex AI Batch Prediction
> - **C)** Embedded in another app → export model, no separate server
> - **D)** Not sure yet — just training for now
>
> If A: Discuss inference framework (vLLM for LLMs, TorchServe, ONNX Runtime, custom FastAPI).

### 10. Inference Cost Estimation

If serving is needed, discuss:
> Expected request volume?
>
> - **A)** Low (< 1000 req/day) → CPU inference on Cloud Run (cheapest, ~$5-20/month)
> - **B)** Medium (1K-100K req/day) → depends on model size; may need GPU
> - **C)** High (> 100K req/day) → dedicated GPU instances, need cost modeling
>
> Provide a rough cost estimate based on the choices made.

### 11. Open-Source Licensing

Ask:
> Are you using pre-trained models? If so, which?
>
> Check the license:
> - **MIT / Apache 2.0** — use commercially, no restrictions
> - **Llama license** — free for most uses, revenue cap
> - **GPL** — must open-source derivative works
> - **Research-only** — cannot use commercially
>
> **Flag licensing risks** if the user plans commercial use with a restrictive model.

---

## Scaffold Spec

```
{project-slug}/
├── src/                        # Main source code
│   ├── train.py                # Training entry point
│   ├── model.py                # Model definition
│   ├── data.py                 # Data loading / preprocessing
│   ├── config.py               # Hyperparameters / config
│   └── serve.py                # Inference server (if serving)
├── tests/
│   └── test_model.py           # Basic model tests
├── data/                       # Local data (gitignored)
├── models/                     # Saved models (gitignored)
├── Dockerfile                  # For Cloud Run deployment (if serving)
├── .dockerignore
├── pyproject.toml              # Python project config (uv/poetry)
├── .python-version             # Python version pin
└── ruff.toml                   # Linter config
```
```

- [ ] **Step 4: Write `startup/routes/internal-tool.md`**

```markdown
# Internal Tool — Architecture Topics

Read this file when the user selected project type: **Internal tool**.

Walk through each topic below **one at a time** via AskUserQuestion.

---

## Common Topics

### 1. Language & Framework

Ask:
> What kind of internal tool?
>
> - **A)** Web dashboard / admin panel → **Next.js** (user's default) or **Retool** (no-code, fast)
> - **B)** CLI / script → depends on what it does (Python, Node, Rust, Go)
> - **C)** Background service / worker → Python or Node
> - **D)** Data pipeline / ETL → Python
>
> **Recommendation for web tools:** If it needs custom logic, Next.js. If it's mostly CRUD on a database, seriously consider Retool — it'll save weeks of development.
>
> Ask: "Is building the UI the point, or is it a means to an end?"

### 2. Database

Ask:
> What data does this tool work with?
>
> - **A)** Existing database (connects to an external DB) → configure connection
> - **B)** Its own database → Firestore (simple) or Cloud SQL Postgres (relational)
> - **C)** Spreadsheets / CSV → Google Sheets API or file import
> - **D)** No persistent data — just processes and outputs

### 3. Authentication

**Important for internal tools** — who should have access?

Ask:
> Who uses this tool?
>
> - **A)** Just me → no auth needed, or basic password
> - **B)** My team (Google Workspace) → **Cloud IAP** (Identity-Aware Proxy — GCP-native, uses Google login, zero code)
> - **C)** Specific people with Google accounts → **Firebase Auth** with allowlist
> - **D)** Behind VPN → no auth needed (VPN is the auth)
>
> **Recommendation:** B (Cloud IAP) for team tools. Zero code auth — GCP handles it at the load balancer level. Anyone with a Google Workspace account in your org gets in automatically.

### 4. Hosting & Scaling

**Default:** Cloud Run.

For internal tools, consider:
- Cloud Run is fine for web dashboards and APIs
- For tools that only you use, `min-instances=0` keeps cost at $0 when idle
- For background workers, Cloud Run Jobs or a long-running Compute Engine instance

### 5. CI/CD

> For internal tools, simpler is better:
>
> - **A)** GitHub Actions (push to main → deploy)
> - **B)** Manual deploy via `gcloud run deploy` (fine for internal tools)
>
> **Recommendation:** B for solo tools, A for team tools.

---

## Internal-Tool-Specific Topics

### 6. Access Control

(Already covered in Authentication above, but dive deeper if B or C was chosen)

If Cloud IAP:
- No code changes needed — IAP sits in front of Cloud Run
- Need to enable IAP API and configure OAuth consent screen
- Can restrict to specific Google accounts or groups

### 7. Build vs Buy

Challenge the user:
> Before we build this — have you looked at existing tools?
>
> - **Retool** — drag-and-drop internal tools, connects to any database/API
> - **Airplane** — similar to Retool, developer-focused
> - **Google AppSheet** — no-code, integrates with Google Sheets
> - **Streamlit** — Python-based dashboards, great for data tools
>
> If the tool is mostly "show data from a database with some filters and actions," Retool might save you weeks. If it needs custom logic, workflows, or a specific UX — build it.

### 8. Data Sources

Ask:
> What data sources does this tool connect to? (Select all that apply)
>
> - **A)** PostgreSQL / MySQL / Cloud SQL
> - **B)** Firestore
> - **C)** Google Sheets
> - **D)** REST APIs
> - **E)** BigQuery
> - **F)** File storage (GCS)
> - **G)** Other — describe
>
> This determines which libraries and GCP APIs to include.

---

## Scaffold Spec

```
{project-slug}/
├── src/app/                    # Next.js App Router
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Main dashboard
│   └── api/health/route.ts     # Health check
├── src/lib/                    # Shared utilities
├── Dockerfile
├── .dockerignore
├── next.config.ts
├── package.json
├── tsconfig.json
└── biome.json
```
```

- [ ] **Step 5: Write `startup/routes/api.md`**

```markdown
# API Service — Architecture Topics

Read this file when the user selected project type: **API service / backend**.

Walk through each topic below **one at a time** via AskUserQuestion.

---

## Common Topics

### 1. Language & Framework

Ask:
> What language for this API?
>
> - **A)** TypeScript + **Express** (simple, mature, huge ecosystem)
> - **B)** TypeScript + **Fastify** (faster than Express, good schema validation)
> - **C)** TypeScript + **Hono** (ultra-light, works on edge, Cloud Run, everywhere)
> - **D)** Python + **FastAPI** (great for data-heavy APIs, auto OpenAPI docs)
> - **E)** Rust + **Axum** (maximum performance, type safety)
> - **F)** Go + **Chi** or **Gin** (fast, simple, great for microservices)
>
> **Recommendation:** Depends on what the API does.
> - CRUD / standard web API → TypeScript + Hono (modern, fast, tiny)
> - Data processing / ML-adjacent → Python + FastAPI
> - Maximum performance → Rust + Axum
> - Microservice in a Go shop → Go + Chi

### 2. Database

Ask:
> What kind of data does this API manage?
>
> - **A)** Relational data (users, orders, products) → **Cloud SQL Postgres**
> - **B)** Document data (flexible schema, nested objects) → **Firestore**
> - **C)** Key-value / cache → **Memorystore (Redis)**
> - **D)** Time-series / analytics → **BigQuery** or **TimescaleDB on Cloud SQL**
> - **E)** No database — stateless API (transforms data, proxies requests)

### 3. Authentication

Ask:
> How do API consumers authenticate?
>
> - **A)** API keys (simple, per-consumer keys)
> - **B)** OAuth 2.0 / JWT tokens (standard, supports third-party auth)
> - **C)** Service-to-service (internal API, use GCP IAM or service account tokens)
> - **D)** No auth (public API)

### 4. Hosting & Scaling

**Default:** Cloud Run. Perfect for APIs — auto-scales, pay-per-request, handles traffic spikes.

**Flag if:**
- WebSocket connections needed → Cloud Run supports them but with a 60-min timeout
- Long-running requests (> 60 min) → consider Compute Engine or GKE

### 5. CI/CD

Same as other routes: GitHub Actions (recommended) or Cloud Build.

---

## API-Specific Topics

### 6. API Spec Format

Ask:
> Do you want an OpenAPI (Swagger) specification from day one?
>
> - **A)** Yes — spec-first development (write the spec, generate code/docs)
> - **B)** Code-first with auto-generated spec (FastAPI does this automatically, or use decorators)
> - **C)** No formal spec — just document in README
>
> **Recommendation:** B for most APIs. FastAPI auto-generates OpenAPI docs. For TypeScript, use Zod schemas that export to OpenAPI.

### 7. Rate Limiting

Ask:
> Does this API need rate limiting?
>
> - **A)** Yes — per-API-key rate limits
> - **B)** Basic — global rate limit to prevent abuse
> - **C)** No — internal API, trusted consumers
>
> If A or B: Discuss implementation:
> - **Redis-backed** (Memorystore) — distributed, production-grade
> - **In-memory** — simple, fine for single-instance Cloud Run
> - **Cloud Armor** — GCP-native, WAF + rate limiting at the load balancer

### 8. API Key Management

If the API uses API keys:
> How should API keys be managed?
>
> - **A)** Simple — generate keys, store in database, validate on each request
> - **B)** API gateway — use GCP API Gateway for key management, throttling, analytics
> - **C)** Third-party — Unkey, Zuplo (managed API key infrastructure)
>
> **Recommendation:** A for most APIs. B if you need GCP-native analytics and throttling.

### 9. Versioning Strategy

Ask:
> How should API versions work?
>
> - **A)** URL path versioning: `/v1/users`, `/v2/users` (most common, clearest)
> - **B)** Header versioning: `Accept: application/vnd.myapi.v1+json`
> - **C)** No versioning (internal API, you control all consumers)
>
> **Recommendation:** A. URL path versioning is the most widely understood and debuggable.

### 10. SDK Generation

Ask:
> Do your API consumers need client libraries (SDKs)?
>
> - **A)** Yes — auto-generate from OpenAPI spec (TypeScript, Python, etc.)
> - **B)** No — consumers use raw HTTP
> - **C)** Maybe later — but design the API with SDK generation in mind
>
> If A: Use openapi-generator or Speakeasy for SDK generation from the OpenAPI spec.

---

## Scaffold Spec

For TypeScript + Hono (example):
```
{project-slug}/
├── src/
│   ├── index.ts                # Entry point + Hono app
│   ├── routes/                 # Route handlers
│   │   └── health.ts           # GET /health
│   ├── middleware/              # Auth, rate limiting, logging
│   └── lib/                    # Shared utilities
├── tests/
│   └── health.test.ts
├── Dockerfile
├── .dockerignore
├── package.json
├── tsconfig.json
└── biome.json
```

For Python + FastAPI:
```
{project-slug}/
├── src/
│   ├── main.py                 # Entry point + FastAPI app
│   ├── routes/                 # Route handlers
│   │   └── health.py           # GET /health
│   ├── middleware/              # Auth, rate limiting
│   └── lib/                    # Shared utilities
├── tests/
│   └── test_health.py
├── Dockerfile
├── .dockerignore
├── pyproject.toml
├── .python-version
└── ruff.toml
```
```

- [ ] **Step 6: Write `startup/routes/cli.md`**

```markdown
# CLI Tool — Architecture Topics

Read this file when the user selected project type: **CLI tool**.

Walk through each topic below **one at a time** via AskUserQuestion.

---

## Common Topics

### 1. Language

This matters more for CLIs than any other project type:

Ask:
> What language for this CLI?
>
> - **A)** **Rust** — single binary, blazing fast, great CLI ecosystem (clap), cross-platform
> - **B)** **Go** — single binary, fast compilation, good CLI ecosystem (cobra), cross-platform
> - **C)** **TypeScript/Node** — fastest to build, npm distribution, requires Node runtime
> - **D)** **Python** — fastest to prototype, rich ecosystem, requires Python runtime
>
> **Recommendation by use case:**
> - Performance-critical / distributed widely → Rust (best UX for end users — single binary, instant startup)
> - Team already uses Go / internal tooling → Go
> - Node ecosystem / rapid prototyping → TypeScript (with bun for single-binary compile)
> - Data processing / scripting → Python

### 2. Database

CLIs rarely need a traditional database:
> Does this CLI need to persist data locally?
>
> - **A)** No — stateless, processes input and outputs
> - **B)** Config file — save user preferences
> - **C)** Local cache / state — SQLite or JSON file
> - **D)** Remote state — connects to an API or database

### 3. Authentication

> Does this CLI need to authenticate with a service?
>
> - **A)** No — standalone tool
> - **B)** API key — stored in config file or env var
> - **C)** OAuth — browser-based login flow (like `gh auth login`)
> - **D)** Service account — GCP or other cloud service

### 4. Hosting & Scaling

CLIs don't need hosting unless they have a backend component:
> Does this CLI have a server component?
>
> - **A)** No — purely client-side tool → no hosting needed
> - **B)** Yes — CLI talks to a backend API → the API part follows the API service route

### 5. CI/CD

> How should CI work?
>
> - **A)** GitHub Actions: lint + test on every push, build releases on tag
> - **B)** Minimal: just lint + test

---

## CLI-Specific Topics

### 6. Distribution

Ask:
> How will users install this CLI?
>
> - **A)** npm (`npm install -g my-cli`) — easiest for Node/Bun CLIs
> - **B)** Homebrew (`brew install my-cli`) — macOS/Linux, needs a tap repo
> - **C)** Binary releases on GitHub (users download from Releases page)
> - **D)** cargo install (`cargo install my-cli`) — for Rust CLIs
> - **E)** Multiple — npm + Homebrew + binary releases
>
> **Recommendation:** For wide distribution, provide binary releases at minimum. Add npm or Homebrew based on audience.

### 7. Config File Format & Location

Ask:
> Where should the CLI store its configuration?
>
> - **A)** XDG standard: `~/.config/{tool-name}/config.toml` (Linux standard, recommended)
> - **B)** Home directory: `~/.{tool-name}/config.json` (simple, visible)
> - **C)** Project-local: `.{tool-name}rc` or `.{tool-name}.json` (per-project config)
> - **D)** No config needed
>
> **File format recommendation:**
> - TOML — most readable, standard for CLI tools
> - JSON — widest tooling support, less human-friendly
> - YAML — readable but whitespace-sensitive (error-prone)

### 8. Update Mechanism

Ask:
> How should the CLI update itself?
>
> - **A)** Self-update command (`my-cli update`) — checks GitHub Releases, downloads new binary
> - **B)** Package manager update (`npm update -g`, `brew upgrade`, `cargo install`)
> - **C)** Manual — user downloads new release
>
> **Recommendation:** A + B. Self-update for binary users, package manager update for package users.

### 9. Shell Completions

Ask:
> Do you want shell completions (tab-complete commands and flags)?
>
> - **A)** Yes — generate for bash, zsh, fish (most CLI frameworks support this)
> - **B)** No — not needed
>
> **Recommendation:** A. It takes 2 lines of code in most CLI frameworks (clap, cobra, yargs) and dramatically improves UX.

---

## Scaffold Spec

For Rust (clap):
```
{project-slug}/
├── src/
│   ├── main.rs                 # Entry point + clap argument parsing
│   ├── cli.rs                  # CLI definition (commands, args)
│   └── commands/               # Command implementations
│       └── mod.rs
├── tests/
│   └── integration.rs
├── Cargo.toml
├── .github/
│   └── workflows/
│       └── release.yml         # Build + publish binaries on tag
└── rustfmt.toml
```

For TypeScript (bun):
```
{project-slug}/
├── src/
│   ├── index.ts                # Entry point
│   ├── cli.ts                  # Argument parsing (yargs/commander)
│   └── commands/               # Command implementations
├── tests/
│   └── cli.test.ts
├── package.json                # bin field configured
├── tsconfig.json
└── biome.json
```
```

- [ ] **Step 7: Commit all route modules**

```bash
git add startup/routes/
git commit -m "feat(startup): add 6 route modules for architecture conversation"
```

---

### Task 4: Main SKILL.md.tmpl — Phase 2 (Architecture Conversation)

**Files:**
- Modify: `startup/SKILL.md.tmpl` (append after Phase 1)

- [ ] **Step 1: Append the architecture conversation phase**

Add this content to the end of `startup/SKILL.md.tmpl`:

```markdown

---

## Phase 2: Architecture Conversation

Now that project basics are collected, have the architecture conversation.

### Step 1: Load the route module

Based on the project type from Phase 1, read the corresponding route file:

```bash
# Read the route module for the chosen project type
# The route file is in the same directory as this skill
```

Read the appropriate file:
- `saas` → Read `startup/routes/saas.md`
- `landing` → Read `startup/routes/landing.md`
- `ai` → Read `startup/routes/ai.md`
- `internal-tool` → Read `startup/routes/internal-tool.md`
- `api` → Read `startup/routes/api.md`
- `cli` → Read `startup/routes/cli.md`

The route module lists the architecture topics and recommendations. Walk through each topic one at a time.

**Important behavior for each topic:**
1. State your recommendation and WHY (architecture advisory mode — be the best software architect)
2. Present 1-2 alternatives with concrete trade-offs
3. Ask the user via AskUserQuestion
4. Wait for sign-off
5. Record the decision (what was chosen, what was rejected, why)

**Durable choices principle:** Recommend technologies and patterns that:
- Have been stable for 3+ years or are clearly the emerging standard
- Have active maintenance and large communities
- Don't lock the user into a specific vendor (except GCP, which is the chosen platform)
- Can scale from prototype to production without rewrites

### Step 2: Service Discovery Scan

After all architecture topics are covered, read the service discovery module:

Read `startup/discovery/service-suggestions.md`

Based on the architecture decisions made, present a smart list of services the user might have forgotten. Ask via AskUserQuestion:

> Based on what we discussed, you might also need these services. Want to add any?
>
> [list from service-suggestions.md, filtered to what's relevant]

Any service the user adds will get:
- Its GCP API enabled (if applicable)
- A section in project.json
- An entry in .env

### Step 3: Architecture Sign-Off

Summarize all architecture decisions:

> **Architecture decisions:**
>
> | Topic | Decision | Rationale |
> |-------|----------|-----------|
> | Language | TypeScript | User's default for web |
> | Framework | Next.js (App Router) | SSR-first, largest ecosystem |
> | Database | Cloud SQL Postgres + Drizzle | Relational data, good TypeScript support |
> | ... | ... | ... |
>
> **Additional services:** [any from discovery scan]
>
> **Looks good? Once confirmed, everything provisions automatically — no more questions.**

Wait for the user to confirm. If they want to change something, go back to that topic.

After confirmation, proceed to Phase 3 (provisioning).
```

- [ ] **Step 2: Commit**

```bash
git add startup/SKILL.md.tmpl
git commit -m "feat(startup): add Phase 2 architecture conversation to main template"
```

---

### Task 5: Service Discovery Module

**Files:**
- Create: `startup/discovery/service-suggestions.md`

- [ ] **Step 1: Write the service discovery module**

```markdown
# Service Discovery — "Did You Forget About..."

Read this file after the architecture conversation is complete. Based on the decisions made, present relevant suggestions the user might have missed.

**Only suggest services that are genuinely relevant** to the project type and architecture. Don't dump the full list — filter it.

---

## Detection Rules

Scan the architecture decisions and project description for these patterns:

### File Uploads / User-Generated Content
**Trigger:** Project description mentions uploads, images, files, documents, media, or avatars.
**Suggest:** Google Cloud Storage + Cloud CDN for serving.
**GCP APIs:** `storage.googleapis.com` (already in always-enable list)

### Feature Flags
**Trigger:** SaaS project type OR multi-tenant architecture.
**Suggest:** PostHog feature flags (already have PostHog — no extra service needed, just enable the feature).
**Why:** Ship to 10% of users first, kill bad features without deploys.

### Async Job Processing
**Trigger:** AI inference, email sending, PDF generation, data processing, webhook handling, any mention of "background jobs" or "queue."
**Suggest:** Google Cloud Tasks (already in always-enable APIs).
**Why:** Don't process heavy work in the request path. Offload to a queue.

### Real-Time Features
**Trigger:** Chat, notifications, live updates, collaboration, multiplayer.
**Suggest:** Cloud Pub/Sub for event fan-out, or WebSocket support in Cloud Run (note: 60-min timeout).
**GCP APIs:** `pubsub.googleapis.com`

### Search
**Trigger:** E-commerce, marketplace, content-heavy sites, documentation.
**Suggest:** Algolia (hosted, fast, expensive) or Meilisearch (self-hosted, cheap) or Postgres full-text search (free, good enough for most).

### Caching / Rate Limiting
**Trigger:** API service type, high-traffic landing page, or any mention of performance.
**Suggest:** Memorystore (Redis) for caching and rate limiting.
**GCP APIs:** `redis.googleapis.com`

### Data Analytics Pipeline
**Trigger:** AI project with large datasets, SaaS with analytics features, any mention of "data pipeline" or "ETL."
**Suggest:** BigQuery for analytics warehouse.
**GCP APIs:** `bigquery.googleapis.com`

### Internationalization (i18n)
**Trigger:** Geography = "Global" or "Europe" (multi-language), or project description mentions "international."
**Suggest:** i18n library setup (next-intl for Next.js), and consider Cloud CDN for global performance.

### Maps / Location
**Trigger:** Project description mentions maps, location, addresses, geocoding, delivery.
**Suggest:** Google Maps Platform APIs.
**GCP APIs:** `maps-backend.googleapis.com`, `places-backend.googleapis.com`, `geocoding-backend.googleapis.com`

### Scheduled Tasks / Cron
**Trigger:** Any SaaS (almost all need scheduled jobs: billing cycles, cleanup, reports).
**Suggest:** Cloud Scheduler (already in always-enable APIs) → triggers Cloud Run endpoint on a cron schedule.
**Why:** Common SaaS need: daily email digests, weekly reports, subscription renewal checks.

### PDF Generation
**Trigger:** Invoices, reports, contracts, receipts.
**Suggest:** Puppeteer/Playwright in a Cloud Run service, or a dedicated PDF library.

---

## Presentation Format

Present suggestions as a checklist:

> Based on your {project-type} with {key-decisions}, you might also need:
>
> - [ ] **Cloud Storage** — you mentioned [trigger]. Store files in GCS with Cloud CDN for fast global delivery.
> - [ ] **Feature flags** — PostHog (already set up) includes feature flags. Ship gradually.
> - [ ] **Background jobs** — Cloud Tasks is already enabled. Use it for [specific use case].
>
> Want to add any of these? (Pick numbers, or "none")

Only show suggestions where you genuinely believe the user might need them. 3-5 suggestions max. Don't overwhelm.
```

- [ ] **Step 2: Commit**

```bash
git add startup/discovery/
git commit -m "feat(startup): add service discovery module"
```

---

### Task 6: Provisioning Modules — GCP, GitHub, DNS

**Files:**
- Create: `startup/provisioning/gcp-project.md`
- Create: `startup/provisioning/github.md`
- Create: `startup/provisioning/dns.md`

- [ ] **Step 1: Write `startup/provisioning/gcp-project.md`**

```markdown
# Provisioning: GCP Project

**Idempotency:** Check if each resource exists before creating. Skip and log if it does.

---

## Step 1: Verify gcloud authentication

```bash
ACCOUNT=$(gcloud auth list --filter=status:ACTIVE --format="value(account)" 2>/dev/null)
echo "ACTIVE_ACCOUNT: $ACCOUNT"
```

If no active account, prompt the user:
> You need to authenticate with Google Cloud first. Please run:
> `! gcloud auth login`
> Then I'll continue.

Wait for the user to authenticate before proceeding.

## Step 2: Create GCP project

```bash
gcloud projects describe {slug} 2>/dev/null && echo "PROJECT_EXISTS" || echo "PROJECT_MISSING"
```

If missing:
```bash
gcloud projects create {slug} --name="{project-name}"
```

If the project ID is taken (globally unique constraint), inform the user and ask for an alternative slug.

## Step 3: Link billing account

```bash
# Detect billing account (user has one billing account)
BILLING=$(gcloud billing accounts list --filter=open=true --format="value(name)" --limit=1)
echo "BILLING_ACCOUNT: $BILLING"

# Check current billing link
gcloud billing projects describe {slug} --format="value(billingAccountName)" 2>/dev/null
```

If not linked:
```bash
gcloud billing projects link {slug} --billing-account=$BILLING
```

## Step 4: Set active project

```bash
gcloud config set project {slug}
```

## Step 5: Enable APIs

Always-enable (batch):
```bash
gcloud services enable \
  run.googleapis.com \
  cloudbuild.googleapis.com \
  storage.googleapis.com \
  dns.googleapis.com \
  artifactregistry.googleapis.com \
  secretmanager.googleapis.com \
  logging.googleapis.com \
  iam.googleapis.com \
  cloudscheduler.googleapis.com \
  cloudtasks.googleapis.com \
  --project={slug}
```

Conditional APIs — enable based on architecture decisions:
- If Cloud SQL chosen: `sqladmin.googleapis.com`
- If Firestore chosen: `firestore.googleapis.com`
- If GPU/AI workloads: `compute.googleapis.com`, `aiplatform.googleapis.com`
- If Cloud Functions: `cloudfunctions.googleapis.com`
- If Pub/Sub: `pubsub.googleapis.com`
- If Redis/Memorystore: `redis.googleapis.com`
- If BigQuery: `bigquery.googleapis.com`
- If Maps: `maps-backend.googleapis.com`, `places-backend.googleapis.com`
- If GA4 API: `analyticsadmin.googleapis.com`
- If Search Console: `searchconsole.googleapis.com`
- If Cloud Monitoring: `monitoring.googleapis.com`
- If IAP (internal tools): `iap.googleapis.com`

Add conditional APIs to the batch command above.

## Step 6: Create service account

```bash
# Check if SA exists
gcloud iam service-accounts describe {slug}-sa@{slug}.iam.gserviceaccount.com 2>/dev/null && echo "SA_EXISTS" || echo "SA_MISSING"
```

If missing:
```bash
gcloud iam service-accounts create {slug}-sa \
  --display-name="{project-name} Service Account" \
  --project={slug}
```

Grant Owner role:
```bash
gcloud projects add-iam-policy-binding {slug} \
  --member="serviceAccount:{slug}-sa@{slug}.iam.gserviceaccount.com" \
  --role="roles/owner" \
  --condition=None
```

## Step 7: Download service account key

```bash
# Check if key already exists
ls ~/.gcloud-keys/{slug}.json 2>/dev/null && echo "KEY_EXISTS" || echo "KEY_MISSING"
```

If missing:
```bash
mkdir -p ~/.gcloud-keys
gcloud iam service-accounts keys create ~/.gcloud-keys/{slug}.json \
  --iam-account={slug}-sa@{slug}.iam.gserviceaccount.com
chmod 600 ~/.gcloud-keys/{slug}.json
```

## Output

Log what was created/skipped:
- GCP project: created / already existed
- Billing: linked / already linked
- APIs: list of APIs enabled
- Service account: created / already existed
- Key: downloaded to ~/.gcloud-keys/{slug}.json / already existed
```

- [ ] **Step 2: Write `startup/provisioning/github.md`**

```markdown
# Provisioning: GitHub Repository

**Idempotency:** Check if repo exists before creating.

---

## Step 1: Initialize git

```bash
ls .git 2>/dev/null && echo "GIT_EXISTS" || echo "GIT_MISSING"
```

If missing:
```bash
git init
```

## Step 2: Check if GitHub repo exists

```bash
gh repo view {org}/{slug} --json name 2>/dev/null && echo "REPO_EXISTS" || echo "REPO_MISSING"
```

## Step 3: Create GitHub repo

If missing:
```bash
gh repo create {org}/{slug} --{visibility} --source . --push
```

If the user specified a personal account (no org), use:
```bash
gh repo create {slug} --{visibility} --source . --push
```

**Note:** `--source .` tells `gh` to use the current directory. `--push` does the initial push.

## Step 4: Set default branch protections (optional)

For production repos, consider:
```bash
# Skip for now — can be configured later via /ship
```

## Output

Log:
- Git: initialized / already existed
- GitHub repo: created at https://github.com/{org}/{slug} / already existed
```

- [ ] **Step 3: Write `startup/provisioning/dns.md`**

```markdown
# Provisioning: Cloud DNS

**Idempotency:** Check if DNS zone exists before creating.

**Skip this step if:** No domain was provided in Phase 1.

---

## Step 1: Check if DNS zone exists

```bash
gcloud dns managed-zones describe {slug}-zone --project={slug} 2>/dev/null && echo "ZONE_EXISTS" || echo "ZONE_MISSING"
```

## Step 2: Create DNS zone

If missing:
```bash
gcloud dns managed-zones create {slug}-zone \
  --dns-name="{domain}." \
  --description="{project-name} DNS zone" \
  --project={slug}
```

## Step 3: Extract nameservers

```bash
gcloud dns managed-zones describe {slug}-zone \
  --project={slug} \
  --format="value(nameServers)"
```

## Step 4: Display nameservers

Present to the user:

> **ACTION REQUIRED — Set nameservers in Namecheap:**
>
> 1. Go to Namecheap → Domain List → {domain} → Custom DNS
> 2. Enter these nameservers:
>    - ns-cloud-a1.googledomains.com
>    - ns-cloud-a2.googledomains.com
>    - ns-cloud-a3.googledomains.com
>    - ns-cloud-a4.googledomains.com
>    (Use the actual nameservers from the output above)
>
> DNS propagation takes 15 minutes to 48 hours.

This is the one expected manual step in the entire /startup flow.

## Output

Log:
- DNS zone: created / already existed
- Nameservers: listed (user must set in Namecheap)
```

- [ ] **Step 4: Commit**

```bash
git add startup/provisioning/gcp-project.md startup/provisioning/github.md startup/provisioning/dns.md
git commit -m "feat(startup): add GCP, GitHub, and DNS provisioning modules"
```

---

### Task 7: Provisioning Modules — Cloud Run, Analytics, PostHog

**Files:**
- Create: `startup/provisioning/cloud-run.md`
- Create: `startup/provisioning/analytics.md`
- Create: `startup/provisioning/posthog.md`

- [ ] **Step 1: Write `startup/provisioning/cloud-run.md`**

```markdown
# Provisioning: Cloud Run + Custom Domain + SSL

**Idempotency:** Check if service exists before deploying.

**Note:** This step happens AFTER scaffold code is generated (Phase 3, Step 7), because we need a Dockerfile and app to deploy.

---

## Step 1: Check if Cloud Run service exists

```bash
gcloud run services describe {slug} --region={region} --project={slug} --format="value(status.url)" 2>/dev/null && echo "SERVICE_EXISTS" || echo "SERVICE_MISSING"
```

## Step 2: Deploy to Cloud Run

If missing or if scaffold was just generated:
```bash
gcloud run deploy {slug} \
  --source . \
  --region={region} \
  --project={slug} \
  --allow-unauthenticated \
  --port=3000 \
  --min-instances=0 \
  --max-instances=10 \
  --memory=512Mi \
  --cpu=1
```

The `--source .` flag uses Cloud Build to build the Dockerfile and deploy.

Adjust `--port` based on the framework:
- Next.js: 3000
- FastAPI: 8000
- Hono/Express: 3000
- Rust (Axum): 8080

## Step 3: Verify deployment

```bash
SERVICE_URL=$(gcloud run services describe {slug} --region={region} --project={slug} --format="value(status.url)")
curl -sf "$SERVICE_URL/api/health" -o /dev/null -w "%{http_code}" 2>/dev/null || curl -sf "$SERVICE_URL/health" -o /dev/null -w "%{http_code}" 2>/dev/null || echo "HEALTH_CHECK_FAILED"
```

## Step 4: Map custom domain (if domain provided)

**Skip if:** No domain was provided.

```bash
gcloud run domain-mappings describe --domain={domain} --region={region} --project={slug} 2>/dev/null && echo "MAPPING_EXISTS" || echo "MAPPING_MISSING"
```

If missing:
```bash
gcloud run domain-mappings create \
  --service={slug} \
  --domain={domain} \
  --region={region} \
  --project={slug}
```

SSL is automatically provisioned by Cloud Run when the domain mapping is created and DNS resolves correctly.

## Step 5: Add DNS records for Cloud Run domain mapping

After creating the domain mapping, Cloud Run outputs required DNS records (typically a CNAME or A record). Add them to Cloud DNS:

```bash
# Get the required DNS records from the domain mapping
gcloud run domain-mappings describe --domain={domain} --region={region} --project={slug} --format=json
```

Add the required records to Cloud DNS using `gcloud dns record-sets create`.

## Output

Log:
- Cloud Run service: deployed to {service-url} / already existed
- Health check: passed / failed
- Domain mapping: created / already existed / skipped (no domain)
- SSL: provisioning (auto by Cloud Run)
```

- [ ] **Step 2: Write `startup/provisioning/analytics.md`**

```markdown
# Provisioning: GA4 + Google Search Console

**Idempotency:** Check if GA4 property exists, check if Search Console is verified.

---

## GA4 Setup

### Step 1: Enable the Analytics Admin API

This should already be enabled in the GCP project step, but verify:
```bash
gcloud services list --enabled --filter="name:analyticsadmin.googleapis.com" --project={slug} --format="value(name)" 2>/dev/null
```

If not enabled:
```bash
gcloud services enable analyticsadmin.googleapis.com --project={slug}
```

### Step 2: Create GA4 property

Use the Google Analytics Admin API via REST (authenticated with gcloud):

```bash
# Get access token
TOKEN=$(gcloud auth print-access-token)

# List existing properties to check for duplicates
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://analyticsadmin.googleapis.com/v1beta/properties?filter=displayName='{project-name}'" \
  | grep -c "properties" || echo "0"
```

If no matching property exists:
```bash
# Create GA4 property
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{project-name}",
    "timeZone": "Europe/Paris",
    "currencyCode": "EUR",
    "industryCategory": "TECHNOLOGY"
  }' \
  "https://analyticsadmin.googleapis.com/v1beta/properties"
```

Extract the property ID from the response (field: `name`, format: `properties/XXXXXXX`).

### Step 3: Create web data stream

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{project-name} Web",
    "defaultUri": "https://{domain}"
  }' \
  "https://analyticsadmin.googleapis.com/v1beta/properties/{property-id}/dataStreams"
```

Extract the measurement ID from the response (`webStreamData.measurementId`, format: `G-XXXXXXXXXX`).

### Step 4: Store values

- Store `measurementId` in `.env` as `NEXT_PUBLIC_GA4_MEASUREMENT_ID`
- Store `propertyId` and `measurementId` in `project.json` under `services.ga4`

**If API calls fail:** Log the error and inform the user:
> GA4 property creation via API failed. Create it manually at https://analytics.google.com and paste the measurement ID. I'll store it for you.

---

## Google Search Console Setup

### Step 1: Add DNS TXT verification record

```bash
# Add TXT record to Cloud DNS for domain verification
gcloud dns record-sets create {domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"google-site-verification={verification-token}"' \
  --zone={slug}-zone \
  --project={slug}
```

**Note:** The verification token comes from the Search Console API. Use the Sites API to initiate verification.

### Step 2: Verify via Search Console API

```bash
TOKEN=$(gcloud auth print-access-token)

# Add site to Search Console
curl -s -X PUT \
  -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/webmasters/v3/sites/https%3A%2F%2F{domain}%2F"
```

### Step 3: Verify ownership

The DNS TXT record should auto-verify. Check:
```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://www.googleapis.com/webmasters/v3/sites/https%3A%2F%2F{domain}%2F"
```

**If verification fails:** DNS may not have propagated yet. Log:
> Search Console: site added, verification pending (DNS propagation may take up to 48h). Re-run /startup to check again.

## Output

Log:
- GA4: property created (ID: {property-id}, measurement: {measurement-id}) / failed (manual setup needed)
- Search Console: verified / pending / failed
```

- [ ] **Step 3: Write `startup/provisioning/posthog.md`**

```markdown
# Provisioning: PostHog

**Idempotency:** Check if PostHog project exists before creating.

Uses PostHog MCP tools (mcp__plugin_posthog_posthog__*).

---

## Step 1: Create PostHog organization

Use the PostHog MCP tool to check existing organizations and create a new one:

Call `mcp__plugin_posthog_posthog__organizations-get` to list existing orgs.

Check if an org matching the project name already exists. If not, a new org will be created when creating the project.

## Step 2: Create PostHog project

Call `mcp__plugin_posthog_posthog__projects-get` to list projects in the org.

If no project matches the slug, create one. PostHog creates projects within organizations — the MCP tools handle this.

**Note:** The PostHog MCP may not have a direct "create org" tool. In that case:
1. Use the existing default organization
2. Create a new project within it named after the project slug
3. Extract the project API key from the project settings

## Step 3: Extract API key and project ID

From the project creation/retrieval response, extract:
- `projectId` — numeric ID
- `apiKey` — the public project API key (starts with `phc_`)
- `host` — the PostHog instance URL (typically `https://us.i.posthog.com` or `https://eu.i.posthog.com`)

## Step 4: Enable error tracking

Call the appropriate PostHog MCP tool to enable error tracking for the project, if available. Otherwise, note that error tracking can be enabled in the PostHog dashboard.

## Step 5: Store values

- Store API key in `.env` as `NEXT_PUBLIC_POSTHOG_KEY`
- Store host in `.env` as `NEXT_PUBLIC_POSTHOG_HOST`
- Store project ID, API key, and host in `project.json` under `services.posthog`

## Output

Log:
- PostHog organization: used existing / created
- PostHog project: created (ID: {project-id}) / already existed
- Error tracking: enabled / manual setup needed
```

- [ ] **Step 4: Commit**

```bash
git add startup/provisioning/cloud-run.md startup/provisioning/analytics.md startup/provisioning/posthog.md
git commit -m "feat(startup): add Cloud Run, analytics, and PostHog provisioning modules"
```

---

### Task 8: Provisioning Modules — Stripe, Email, Monitoring, CI/CD

**Files:**
- Create: `startup/provisioning/stripe.md`
- Create: `startup/provisioning/email.md`
- Create: `startup/provisioning/monitoring.md`
- Create: `startup/provisioning/cicd.md`

- [ ] **Step 1: Write `startup/provisioning/stripe.md`**

```markdown
# Provisioning: Stripe (Conditional — Paid Projects Only)

**Skip this step if:** Project is free (no Stripe).

Uses Stripe MCP tools (mcp__plugin_stripe_stripe__*). Always in **test mode**.

---

## Step 1: Verify Stripe account

Call `mcp__plugin_stripe_stripe__get_stripe_account_info` to verify the Stripe connection is active.

If not connected, inform the user:
> Stripe MCP is not connected. Connect it in your Claude Code settings, then re-run /startup.

## Step 2: Create product

Call `mcp__plugin_stripe_stripe__create_product` with:
- `name`: the project name
- `description`: the project description

Store the `product.id` (format: `prod_xxx`).

## Step 3: Create price

Based on the billing model from the architecture conversation:

Call `mcp__plugin_stripe_stripe__create_price` with:
- `product`: the product ID from Step 2
- `unit_amount`: price in cents (ask user during architecture if not decided)
- `currency`: `eur` (or based on geography)
- `recurring.interval`: `month` or `year` (based on architecture decision)

For seat-based pricing, set `billing_scheme: "per_unit"`.

Store the `price.id` (format: `price_xxx`).

## Step 4: Create webhook endpoint

Call `mcp__plugin_stripe_stripe__create_payment_link` — actually, for webhooks, use the Stripe API directly via curl since MCP may not have a webhook creation tool:

The webhook endpoint URL is: `https://{domain}/api/webhooks/stripe`
(or Cloud Run URL if no domain)

Events to listen for:
- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`

**Note:** If the Stripe MCP doesn't support webhook endpoint creation, log this as a manual step:
> Stripe webhook: Create manually at https://dashboard.stripe.com/test/webhooks
> Endpoint URL: https://{domain}/api/webhooks/stripe
> Events: checkout.session.completed, customer.subscription.*, invoice.payment_*

## Step 5: Store values

- Store in `.env`:
  - `STRIPE_SECRET_KEY` — from Stripe dashboard (test mode key: `sk_test_xxx`)
  - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` — `pk_test_xxx`
  - `STRIPE_WEBHOOK_SECRET` — `whsec_xxx` (from webhook creation)
- Store in `project.json` under `services.stripe`:
  - `mode`: "test"
  - `productId`: from Step 2
  - `priceId`: from Step 3
  - `webhookEndpointId`: from Step 4 (if created)
  - `customerPortalEnabled`: true

**Important:** The Stripe MCP tools work in the context of the connected Stripe account. The API keys themselves may need to be copied from the Stripe dashboard. Inform the user:
> Stripe product and price created in test mode. You'll need to copy your test API keys from https://dashboard.stripe.com/test/apikeys into your .env file.

## Output

Log:
- Stripe product: created (ID: {product-id})
- Stripe price: created (ID: {price-id})
- Stripe webhook: created / manual setup needed
- API keys: user must copy from Stripe dashboard
```

- [ ] **Step 2: Write `startup/provisioning/email.md`**

```markdown
# Provisioning: Resend (Conditional — Email Projects Only)

**Skip this step if:** Project doesn't need email.

---

## Step 1: Store Resend API key

Ask the user for their Resend API key via AskUserQuestion:
> Enter your Resend API key (from https://resend.com/api-keys):

Store in `.env` as `RESEND_API_KEY`.

## Step 2: Add domain DNS records to Cloud DNS

Resend requires DNS records for domain verification (DKIM, SPF, DMARC). These records are specific to the sending domain.

After the user provides the API key, use the Resend API to get the required DNS records:

```bash
TOKEN="{resend-api-key}"

# Add domain to Resend
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "{domain}"}' \
  "https://api.resend.com/domains"
```

From the response, extract the DNS records and add them to Cloud DNS:

```bash
# Add DKIM records (typically 3 CNAME records)
gcloud dns record-sets create {dkim-host}. \
  --type=CNAME \
  --ttl=300 \
  --rrdatas="{dkim-value}." \
  --zone={slug}-zone \
  --project={slug}

# Add SPF record (TXT)
gcloud dns record-sets create {domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"v=spf1 include:amazonses.com ~all"' \
  --zone={slug}-zone \
  --project={slug}

# Add DMARC record (TXT)
gcloud dns record-sets create _dmarc.{domain}. \
  --type=TXT \
  --ttl=300 \
  --rrdatas='"v=DMARC1; p=none;"' \
  --zone={slug}-zone \
  --project={slug}
```

**Note:** Exact DNS records come from the Resend API response. The above are examples — use the actual values from the response.

## Step 3: Create welcome email template

Use the Resend API to create a basic welcome email template:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "welcome",
    "subject": "Welcome to {project-name}",
    "html": "<h1>Welcome to {project-name}!</h1><p>Thanks for signing up. We'\''re excited to have you.</p>"
  }' \
  "https://api.resend.com/emails"
```

**Note:** Resend may not have a template creation API. In that case, scaffold the template as a code file in the project instead (e.g., `src/emails/welcome.tsx` using React Email).

## Step 4: Store values

- `.env`: `RESEND_API_KEY`
- `project.json` under `services.resend`:
  - `sendingDomain`: the domain
  - `domainVerified`: true/false (check after DNS records are added)

## Output

Log:
- Resend domain: added + DNS records created in Cloud DNS
- Domain verification: pending (DNS propagation)
- Welcome template: created in code / via API
```

- [ ] **Step 3: Write `startup/provisioning/monitoring.md`**

```markdown
# Provisioning: Monitoring (Uptime Checks + Alerts)

---

## Step 1: Enable Cloud Monitoring API

```bash
gcloud services enable monitoring.googleapis.com --project={slug}
```

## Step 2: Create notification channel (email)

```bash
# Get user's email from gcloud auth
EMAIL=$(gcloud auth list --filter=status:ACTIVE --format="value(account)")

# Create email notification channel
gcloud alpha monitoring channels create \
  --type=email \
  --display-name="{project-name} Alerts" \
  --channel-labels=email_address=$EMAIL \
  --project={slug}
```

Extract the channel ID from the output.

**Note:** If `gcloud alpha monitoring channels create` is not available, use the REST API:

```bash
TOKEN=$(gcloud auth print-access-token)
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "email",
    "displayName": "{project-name} Alerts",
    "labels": {"email_address": "'$EMAIL'"}
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/notificationChannels"
```

## Step 3: Create uptime check

```bash
TOKEN=$(gcloud auth print-access-token)

# Target: domain if available, else Cloud Run URL
CHECK_HOST="{domain or cloud-run-url}"

curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug}-uptime",
    "monitoredResource": {
      "type": "uptime_url",
      "labels": {"host": "'$CHECK_HOST'", "project_id": "{slug}"}
    },
    "httpCheck": {
      "path": "/api/health",
      "port": 443,
      "useSsl": true,
      "requestMethod": "GET"
    },
    "period": "300s",
    "timeout": "10s"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/uptimeCheckConfigs"
```

## Step 4: Create alert policies

### 5xx Error Rate Alert

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug} 5xx Error Rate",
    "conditions": [{
      "displayName": "5xx rate > 1%",
      "conditionThreshold": {
        "filter": "resource.type=\"cloud_run_revision\" AND resource.labels.service_name=\"{slug}\" AND metric.type=\"run.googleapis.com/request_count\" AND metric.labels.response_code_class=\"5xx\"",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 0.01,
        "duration": "300s",
        "aggregations": [{
          "alignmentPeriod": "300s",
          "perSeriesAligner": "ALIGN_RATE"
        }]
      }
    }],
    "notificationChannels": ["{channel-id}"],
    "combiner": "OR"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/alertPolicies"
```

### Latency Alert

```bash
curl -s -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "{slug} High Latency",
    "conditions": [{
      "displayName": "p95 latency > 2s",
      "conditionThreshold": {
        "filter": "resource.type=\"cloud_run_revision\" AND resource.labels.service_name=\"{slug}\" AND metric.type=\"run.googleapis.com/request_latencies\"",
        "comparison": "COMPARISON_GT",
        "thresholdValue": 2000,
        "duration": "300s",
        "aggregations": [{
          "alignmentPeriod": "300s",
          "perSeriesAligner": "ALIGN_PERCENTILE_95"
        }]
      }
    }],
    "notificationChannels": ["{channel-id}"],
    "combiner": "OR"
  }' \
  "https://monitoring.googleapis.com/v3/projects/{slug}/alertPolicies"
```

## Output

Log:
- Notification channel: created (email: {email})
- Uptime check: created on {host}/api/health
- Alert policy: 5xx error rate > 1% → email
- Alert policy: p95 latency > 2s → email
```

- [ ] **Step 4: Write `startup/provisioning/cicd.md`**

```markdown
# Provisioning: CI/CD Pipeline

Set up the CI/CD pipeline based on the architecture decision.

---

## GitHub Actions (default)

If GitHub Actions was chosen, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

env:
  PROJECT_ID: {slug}
  REGION: {region}
  SERVICE: {slug}

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - id: auth
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - uses: google-github-actions/setup-gcloud@v2

      - name: Deploy to Cloud Run
        run: |
          gcloud run deploy $SERVICE \
            --source . \
            --region $REGION \
            --project $PROJECT_ID \
            --allow-unauthenticated \
            --quiet
```

**Setup required:**
1. Add `GCP_SA_KEY` as a GitHub Actions secret (base64-encoded service account key)
2. The key is at `~/.gcloud-keys/{slug}.json`

```bash
# Encode the key for GitHub Actions
cat ~/.gcloud-keys/{slug}.json | base64 | gh secret set GCP_SA_KEY --repo={org}/{slug}
```

Also create a lint/test workflow `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      # Framework-specific setup (bun, node, python, rust, etc.)
      # Lint + test commands from CLAUDE.md
```

Adapt the CI workflow to the chosen framework:
- Next.js / TypeScript → `bun install && bun run lint && bun test`
- Python → `uv sync && ruff check . && pytest`
- Rust → `cargo fmt --check && cargo clippy && cargo test`

## Cloud Build (alternative)

If Cloud Build was chosen instead:

Create `cloudbuild.yaml`:

```yaml
steps:
  - name: 'gcr.io/cloud-builders/gcloud'
    args:
      - 'run'
      - 'deploy'
      - '{slug}'
      - '--source=.'
      - '--region={region}'
      - '--allow-unauthenticated'
      - '--quiet'
```

Set up Cloud Build trigger:
```bash
gcloud builds triggers create github \
  --repo-name={slug} \
  --repo-owner={org} \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml \
  --project={slug}
```

## Output

Log:
- CI/CD: GitHub Actions / Cloud Build configured
- Workflows created: deploy.yml, ci.yml
- GitHub secret: GCP_SA_KEY set (if GitHub Actions)
```

- [ ] **Step 5: Commit**

```bash
git add startup/provisioning/stripe.md startup/provisioning/email.md startup/provisioning/monitoring.md startup/provisioning/cicd.md
git commit -m "feat(startup): add Stripe, email, monitoring, and CI/CD provisioning modules"
```

---

### Task 9: Generator Modules

**Files:**
- Create: `startup/generators/readme.md`
- Create: `startup/generators/project-json.md`
- Create: `startup/generators/claude-md.md`
- Create: `startup/generators/adr.md`
- Create: `startup/generators/gitignore.md`
- Create: `startup/generators/scaffold.md`

- [ ] **Step 1: Write `startup/generators/readme.md`**

```markdown
# Generator: README.md

Generate a beautiful, polished README that makes the project look professional from day one.

---

## Structure

Every README follows this exact structure:

### 1. ASCII Art Header

Generate custom ASCII art from the project name using block letters. Use a simple, clean font style. Example for "MyApp":

```
 __  __          _
|  \/  |        / \   _ __  _ __
| |\/| |  _   / _ \ | '_ \| '_ \
| |  | | | |_/ ___ \| |_) | |_) |
|_|  |_|  \__/_/   \_\ .__/| .__/
                      |_|   |_|
```

Keep it readable. If the name is too long (> 10 chars), use a simpler style or abbreviate.

### 2. One-Paragraph Description

One paragraph that **sells** the project. Not technical — explain what it does for the user, not how it works. Example:

> MyApp is a collaborative project management tool that lets teams plan, track, and ship software without drowning in status meetings. Real-time updates, smart prioritization, and zero overhead.

### 3. Badges

```markdown
![Build Status](https://github.com/{org}/{slug}/actions/workflows/ci.yml/badge.svg)
![License](https://img.shields.io/badge/license-MIT-blue.svg)
```

Add relevant badges based on project type. Only include badges that have actual backing data.

### 4. Screenshot / Demo Placeholder

```markdown
<!-- TODO: Add screenshot or demo GIF -->
<p align="center">
  <img src="docs/screenshot.png" alt="Screenshot" width="600">
</p>
```

### 5. Quick Start

Exactly 3 steps. No more.

```markdown
## Quick Start

\```bash
# 1. Clone
git clone https://github.com/{org}/{slug}.git && cd {slug}

# 2. Install
bun install  # or: pip install -e . / cargo build

# 3. Run
bun dev  # or: python -m src.main / cargo run
\```
```

### 6. Architecture Overview

Brief description + simple diagram (use code block diagram or link to architecture doc):

```markdown
## Architecture

{2-3 sentence description of how the system works}

Built with {framework} on Google Cloud Run. Data stored in {database}.
```

### 7. Tech Stack Table

```markdown
## Tech Stack

| Component | Technology |
|-----------|-----------|
| Framework | Next.js 15 |
| Language | TypeScript |
| Database | Cloud SQL (Postgres) |
| Hosting | Google Cloud Run |
| Analytics | GA4 + PostHog |
| Payments | Stripe |
```

### 8. Development Setup

Detailed local dev setup for contributors:
- Prerequisites (Node, Bun, Python, Rust, etc.)
- Environment variables (reference .env.example)
- Database setup (if applicable)
- How to run tests
- How to lint

### 9. Deployment

Brief deployment instructions referencing CI/CD setup:
```markdown
## Deployment

Push to `main` triggers automatic deployment via GitHub Actions to Google Cloud Run.

Manual deploy: `gcloud run deploy {slug} --source . --region {region}`
```

### 10. Contributing

```markdown
## Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request
```

### 11. License

```markdown
## License

MIT License. See [LICENSE](LICENSE) for details.
```

---

## Quality Rules

- **No placeholder text.** Every section should have real content based on the project.
- **Sell, don't describe.** The README is the project's landing page. Make people want to use it.
- **Be specific.** "A web app" → "A real-time collaborative task manager for engineering teams."
- **Keep it scannable.** Headers, tables, code blocks. No walls of text.
- **ASCII art must be clean.** If the font doesn't render well for the project name, use a simpler font or just a large text header.
```

- [ ] **Step 2: Write `startup/generators/project-json.md`**

```markdown
# Generator: project.json + PROJECT.md

---

## project.json Schema

The source of truth for all project metadata. Committed to git. No secrets.

```json
{
  "name": "{project-name}",
  "slug": "{slug}",
  "description": "{description}",
  "type": "{saas|landing|ai|internal-tool|api|cli}",
  "created": "{YYYY-MM-DD}",

  "architecture": {
    "language": "{typescript|python|rust|go}",
    "framework": "{next.js|fastapi|axum|hono|astro|etc.}",
    "database": "{cloud-sql-postgres|firestore|none|etc.}",
    "orm": "{drizzle|prisma|sqlalchemy|none}",
    "auth": "{firebase-auth|clerk|nextauth|none|etc.}",
    "hosting": "cloud-run",
    "cicd": "{github-actions|cloud-build}",
    "linter": "{biome|ruff|rustfmt|etc.}"
  },

  "gcp": {
    "projectId": "{slug}",
    "region": "{region}",
    "serviceAccount": "{slug}-sa@{slug}.iam.gserviceaccount.com",
    "cloudRunService": "{slug}",
    "dnsZone": "{slug}-zone"
  },

  "domain": "{domain or null}",

  "github": {
    "org": "{org}",
    "repo": "{slug}",
    "visibility": "{public|private}"
  },

  "services": {
    "ga4": {
      "propertyId": "{id or null}",
      "measurementId": "{G-XXXXXXXXXX or null}",
      "dataStreamId": "{id or null}"
    },
    "searchConsole": {
      "verified": false
    },
    "posthog": {
      "orgId": "{org-id}",
      "projectId": "{project-id}",
      "host": "{posthog-host}"
    },
    "stripe": null,
    "resend": null,
    "errorTracking": {
      "provider": "posthog",
      "enabled": true
    }
  },

  "monitoring": {
    "uptimeCheck": "{slug}-uptime",
    "alertPolicies": ["5xx-rate", "latency"]
  },

  "environments": ["{env-list}"]
}
```

Only include `services.stripe` if the project is paid. Only include `services.resend` if email is needed. Set to `null` if not applicable.

---

## PROJECT.md Generation

Generate from project.json. Format:

```markdown
# {project-name}

> {description}

## Quick Links

| Service | Link |
|---------|------|
| GitHub | https://github.com/{org}/{slug} |
| GCP Console | https://console.cloud.google.com/home/dashboard?project={slug} |
| Cloud Run | {cloud-run-url} |
| GA4 | https://analytics.google.com/analytics/web/#/p{property-id} |
| PostHog | https://app.posthog.com/project/{project-id} |
| Search Console | https://search.google.com/search-console?resource_id=sc-domain:{domain} |
| Stripe | https://dashboard.stripe.com/test/products/{product-id} |
| Domain | https://{domain} |

## Architecture

- **Type:** {type}
- **Stack:** {language} + {framework}
- **Database:** {database}
- **Auth:** {auth}
- **Hosting:** Cloud Run ({region})
- **CI/CD:** {cicd}

## Service Status

| Service | Status |
|---------|--------|
| GCP Project | {slug} |
| Cloud Run | Deployed |
| GA4 | {measurement-id or "Not configured"} |
| PostHog | Project #{project-id} |
| Stripe | {Test mode or "Not configured"} |
| Resend | {domain or "Not configured"} |
| DNS | {domain or "No domain"} |
| Monitoring | Uptime check active |
```

Only include rows for services that are configured. Skip null services.
```

- [ ] **Step 3: Write `startup/generators/claude-md.md`**

```markdown
# Generator: CLAUDE.md

Seed the project's CLAUDE.md with all context from /startup.

---

## Template

```markdown
# {project-name}

{description}

## Stack

- **Language:** {language}
- **Framework:** {framework}
- **Database:** {database} {+ ORM if applicable}
- **Auth:** {auth}
- **Hosting:** Google Cloud Run ({region})
- **CI/CD:** {cicd}

## Commands

\```bash
{dev-command}         # local development server
{build-command}       # production build
{test-command}        # run tests
{lint-command}        # lint + format
{deploy-command}      # deploy to Cloud Run
\```

## Project Config

- **Project metadata:** `project.json` (source of truth, committed)
- **Secrets:** `.env` (gitignored) — copy from `.env.example` for local dev
- **GCP service account key:** `~/.gcloud-keys/{slug}.json`
- **Production secrets:** GCP Secret Manager (project: {slug})
- **Architecture decisions:** `docs/decisions/`

## GCP

- **Project ID:** {slug}
- **Region:** {region}
- **Service account:** {slug}-sa@{slug}.iam.gserviceaccount.com
- **Cloud Run service:** {slug}

## Deploy Configuration (configured by /startup)

- Platform: Google Cloud Run
- Production URL: https://{domain or cloud-run-url}
- Deploy trigger: {GitHub Actions push to main / Cloud Build / manual}
- Health check: https://{domain or cloud-run-url}/api/health
- Project type: {type}

## Conventions

{List any conventions decided during the architecture conversation. Examples:}
{- Multi-tenant: shared database with tenant_id column}
{- API versioning: URL path (/v1/...)}
{- State management: React Server Components, minimal client state}
```

## Commands by Framework

Fill in the commands based on the framework:

| Framework | Dev | Build | Test | Lint | Deploy |
|-----------|-----|-------|------|------|--------|
| Next.js | `bun dev` | `bun run build` | `bun test` | `bun run lint` | `gcloud run deploy {slug} --source .` |
| FastAPI | `uv run uvicorn src.main:app --reload` | `docker build -t {slug} .` | `uv run pytest` | `uv run ruff check .` | `gcloud run deploy {slug} --source .` |
| Hono | `bun dev` | `bun run build` | `bun test` | `bun run lint` | `gcloud run deploy {slug} --source .` |
| Rust (Axum) | `cargo run` | `cargo build --release` | `cargo test` | `cargo fmt --check && cargo clippy` | `gcloud run deploy {slug} --source .` |
| Astro | `bun dev` | `bun run build` | `bun test` | `bun run lint` | `gcloud run deploy {slug} --source .` |
```

- [ ] **Step 4: Write `startup/generators/adr.md`**

```markdown
# Generator: Architecture Decision Records

Create `docs/decisions/001-initial-architecture.md` from the architecture conversation.

---

## Format

```markdown
# ADR 001: Initial Architecture

**Date:** {YYYY-MM-DD}
**Status:** Accepted
**Context:** New {type} project — {description}

## Decisions

### {Topic 1: e.g., Language & Framework}

**Chosen:** {decision}
**Alternatives considered:**
- {alternative 1} — {why rejected}
- {alternative 2} — {why rejected}
**Rationale:** {why this was chosen, including user's reasoning}

### {Topic 2: e.g., Database}

**Chosen:** {decision}
**Alternatives considered:**
- {alternative 1} — {why rejected}
**Rationale:** {reasoning}

{... one section per architecture decision ...}

## Additional Services

{List any services added from the discovery scan, with rationale}

## Consequences

- {Positive consequence 1}
- {Positive consequence 2}
- {Risk or trade-off 1}
- {Things to revisit later}
```

## Rules

- **One section per decision.** Don't combine topics.
- **Include user's reasoning.** If the user said "I want Postgres because I know SQL," record that.
- **Record what was rejected and why.** Future-you will thank present-you.
- **Be honest about trade-offs.** Every decision has downsides. Name them.
- **Date it.** ADRs are time-stamped. Decisions can be revisited.
```

- [ ] **Step 5: Write `startup/generators/gitignore.md`**

```markdown
# Generator: .gitignore

Generate a .gitignore tailored to the chosen stack.

---

## Always Include (every project)

```gitignore
# Environment
.env
.env.local
.env.*.local

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# GCP keys (should be in ~/.gcloud-keys/, but just in case)
*.json.key
service-account*.json
```

## By Language/Framework

### TypeScript / Next.js

```gitignore
node_modules/
.next/
out/
dist/
*.tsbuildinfo
.turbo/
```

### Python

```gitignore
__pycache__/
*.pyc
*.pyo
.venv/
venv/
*.egg-info/
dist/
build/
.ruff_cache/
wandb/
*.pt
*.bin
*.safetensors
```

### Rust

```gitignore
target/
Cargo.lock  # only for libraries; keep for binaries
```

### Go

```gitignore
bin/
vendor/
```

## By Service

### Docker

```gitignore
# Don't ignore Dockerfile or .dockerignore
```

### Terraform (if used)

```gitignore
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
```

---

Combine the "Always Include" section with the relevant language/framework and service sections.
```

- [ ] **Step 6: Write `startup/generators/scaffold.md`**

```markdown
# Generator: Code Scaffold

Generate a working skeleton based on the project route and architecture decisions.

**Hard rule:** The scaffold MUST:
1. Build successfully
2. Run locally
3. Deploy to Cloud Run
4. Pass a health check at `/api/health` or `/health`

**No business logic.** Just a working shell with:
- Proper project structure
- Health endpoint
- Dockerfile optimized for Cloud Run
- Linter/formatter configured
- GA4 tracking code installed (if web project)
- PostHog SDK installed (if web project)
- Stripe webhook handler (if paid)
- Email utility (if Resend)

---

## Dockerfile Pattern (all projects)

Use multi-stage builds for smaller images:

### Next.js
```dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json bun.lockb* ./
RUN npm install --global bun && bun install --frozen-lockfile

FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm install --global bun && bun run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### Python (FastAPI)
```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY pyproject.toml ./
RUN pip install uv && uv sync --no-dev

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/.venv ./.venv
COPY src/ ./src/
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8000
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Rust (Axum)
```dockerfile
FROM rust:1.77 AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src/ ./src/
RUN cargo build --release

FROM debian:bookworm-slim
COPY --from=builder /app/target/release/{slug} /usr/local/bin/
EXPOSE 8080
CMD ["{slug}"]
```

---

## Health Endpoint Pattern

Every scaffold includes a health endpoint:

### Next.js (App Router)
`src/app/api/health/route.ts`:
```typescript
export function GET() {
  return Response.json({ status: 'ok', timestamp: new Date().toISOString() });
}
```

### FastAPI
`src/routes/health.py`:
```python
from fastapi import APIRouter
router = APIRouter()

@router.get("/health")
def health():
    return {"status": "ok", "timestamp": __import__("datetime").datetime.now().isoformat()}
```

### Hono
`src/routes/health.ts`:
```typescript
import { Hono } from 'hono';
const app = new Hono();
app.get('/health', (c) => c.json({ status: 'ok', timestamp: new Date().toISOString() }));
export default app;
```

---

## Analytics Installation

### GA4 (Next.js)
In `src/app/layout.tsx`, add the GA4 script:
```tsx
import Script from 'next/script';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <head>
        <Script src={`https://www.googletagmanager.com/gtag/js?id=${process.env.NEXT_PUBLIC_GA4_MEASUREMENT_ID}`} strategy="afterInteractive" />
        <Script id="ga4" strategy="afterInteractive">
          {`window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', '${process.env.NEXT_PUBLIC_GA4_MEASUREMENT_ID}');`}
        </Script>
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### PostHog (Next.js)
Create `src/lib/posthog.ts`:
```typescript
import posthog from 'posthog-js';

export function initPostHog() {
  if (typeof window !== 'undefined' && process.env.NEXT_PUBLIC_POSTHOG_KEY) {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://us.i.posthog.com',
    });
  }
}
```

For Python/Rust/Go APIs: install the PostHog server-side SDK instead.

---

## Linter Configuration

### Biome (TypeScript/JavaScript)
`biome.json`:
```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2
  }
}
```

### Ruff (Python)
`ruff.toml`:
```toml
line-length = 100
target-version = "py312"

[lint]
select = ["E", "F", "I", "N", "W", "UP"]
```

### Rustfmt (Rust)
`rustfmt.toml`:
```toml
edition = "2021"
max_width = 100
```
```

- [ ] **Step 7: Commit all generators**

```bash
git add startup/generators/
git commit -m "feat(startup): add all generator modules (README, project.json, CLAUDE.md, ADR, gitignore, scaffold)"
```

---

### Task 10: Main SKILL.md.tmpl — Phases 3-4 (Provisioning + File Gen + Ship + Summary)

**Files:**
- Modify: `startup/SKILL.md.tmpl` (append after Phase 2)

- [ ] **Step 1: Append Phase 3 (provisioning) to the main template**

Add this content to the end of `startup/SKILL.md.tmpl`:

```markdown

---

## Phase 3: Provisioning

**All provisioning runs automatically.** No per-step confirmations. Errors are collected and reported at the end.

Read each provisioning module and execute the steps. For each module:
1. Read the module file
2. Execute each step
3. Log success or failure
4. Continue to the next module regardless of failures

### Provisioning Order

Execute in this order (dependencies matter):

1. **GCP Project** — Read `startup/provisioning/gcp-project.md` and execute.
   Must succeed before anything else. If GCP auth fails, stop here and prompt the user.

2. **GitHub** — Read `startup/provisioning/github.md` and execute.
   Can run independently of GCP.

3. **Cloud DNS** — Read `startup/provisioning/dns.md` and execute.
   Requires GCP project. Skip if no domain.

4. **Scaffold Code** — Read `startup/generators/scaffold.md` and generate the working skeleton.
   Uses architecture decisions from Phase 2. Also read the route module's "Scaffold Spec" section for the project structure.

   After scaffolding:
   - Read `startup/generators/gitignore.md` and generate `.gitignore`
   - Install dependencies (bun install, uv sync, cargo build, etc.)
   - Verify the scaffold builds: run the build command
   - Verify the health endpoint works locally if possible

5. **Cloud Run** — Read `startup/provisioning/cloud-run.md` and execute.
   Requires scaffold code (needs Dockerfile). Deploys the skeleton to Cloud Run.

6. **GA4 + Search Console** — Read `startup/provisioning/analytics.md` and execute.
   Requires GCP project. Search Console requires Cloud DNS (for DNS verification).

7. **PostHog** — Read `startup/provisioning/posthog.md` and execute.
   Independent of GCP.

8. **Stripe** — Read `startup/provisioning/stripe.md` and execute.
   Only if paid. Independent of GCP.

9. **Resend** — Read `startup/provisioning/email.md` and execute.
   Only if email. Requires Cloud DNS (for domain verification records).

10. **Monitoring** — Read `startup/provisioning/monitoring.md` and execute.
    Requires Cloud Run service to be deployed.

11. **CI/CD** — Read `startup/provisioning/cicd.md` and execute.
    Requires GitHub repo. Creates workflow files and sets secrets.

### Error Collection

Keep a running log of all provisioning results:

```
PROVISIONING_LOG:
  gcp-project: SUCCESS
  github: SUCCESS
  dns: SUCCESS
  scaffold: SUCCESS
  cloud-run: SUCCESS
  ga4: FAILED — "Analytics Admin API returned 403"
  search-console: SKIPPED — "No domain"
  posthog: SUCCESS
  stripe: SKIPPED — "Free project"
  resend: SKIPPED — "No email"
  monitoring: SUCCESS
  cicd: SUCCESS
```

---

## Phase 4: File Generation

After provisioning, generate the project metadata files.

### Step 1: project.json

Read `startup/generators/project-json.md`. Generate `project.json` with all values collected during provisioning (IDs, keys, etc.). Write to the project root.

### Step 2: PROJECT.md

Generate `PROJECT.md` from `project.json` using the format in `startup/generators/project-json.md`. Write to the project root.

### Step 3: .env and .env.example

Generate `.env` with all secrets collected during provisioning. Generate `.env.example` with the same structure but empty values. Write both to the project root.

### Step 4: CLAUDE.md

Read `startup/generators/claude-md.md`. Generate `CLAUDE.md` using all project context. Write to the project root.

If a CLAUDE.md already exists (from a re-run), merge new content — don't overwrite user additions.

### Step 5: ADR

Read `startup/generators/adr.md`. Generate `docs/decisions/001-initial-architecture.md` from all architecture decisions. Create the `docs/decisions/` directory.

### Step 6: README.md

Read `startup/generators/readme.md`. Generate a beautiful README with ASCII art, all sections populated. Write to the project root.

### Step 7: Legal pages (if requested)

If the user requested legal pages:
- Create `src/app/privacy/page.tsx` (or equivalent) with a TODO placeholder
- Create `src/app/terms/page.tsx` (or equivalent) with a TODO placeholder
- Include basic structure and a note: "Replace this with your actual privacy policy / terms of service"

---

## Phase 5: Ship

### Step 1: First commit and push

```bash
git add -A
git commit -m "Initial project setup via /startup"
git push -u origin main
```

If the repo was already created and pushed in the GitHub provisioning step, this may just be an update push:
```bash
git add -A
git commit -m "Complete project setup via /startup"
git push
```

### Step 2: Summary Report

Present the final summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 {project-name} is ready
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GitHub:          https://github.com/{org}/{slug}
GCP Console:     https://console.cloud.google.com/home/dashboard?project={slug}
Cloud Run:       {cloud-run-url}
Domain:          https://{domain} {(pending DNS propagation) if applicable}

GA4:             https://analytics.google.com/analytics/web/#/p{property-id}
Search Console:  https://search.google.com/search-console?resource_id=sc-domain:{domain}
PostHog:         https://app.posthog.com/project/{posthog-project-id}
{Stripe:         https://dashboard.stripe.com/test/products/{product-id}}
{Resend:         https://resend.com/domains}

Local dev:       {dev-command}
Build:           {build-command}
Test:            {test-command}
Deploy:          {deploy-command}

Manual steps remaining:
  {1. Set nameservers in Namecheap: ns1... ns2... ns3... ns4...}
  {2. Copy Stripe API keys from dashboard to .env}
  {3. Any other manual steps from failed provisioning}

{PROVISIONING FAILURES (if any):
  - {service}: {error} — re-run /startup to retry}

Files created:
  project.json          — project metadata (committed)
  PROJECT.md            — human-readable project card (committed)
  .env                  — secrets (gitignored)
  .env.example          — template (committed)
  CLAUDE.md             — AI assistant context (committed)
  docs/decisions/001-initial-architecture.md — architecture decisions
  README.md             — project README with ASCII art
  .gitignore            — stack-tailored
  {.github/workflows/}  — CI/CD pipelines
```

Include only relevant lines (skip Stripe if free, skip domain if none, etc.).

### Step 3: Next steps suggestion

> **What's next?**
>
> - `/office-hours` — brainstorm features before building
> - `/design-consultation` — create a design system (colors, typography, spacing)
> - Start coding — your scaffold is deployed and ready
```

- [ ] **Step 2: Commit**

```bash
git add startup/SKILL.md.tmpl
git commit -m "feat(startup): add Phases 3-5 (provisioning, file generation, ship, summary)"
```

---

### Task 11: Generate SKILL.md and Validate

**Files:**
- Generate: `startup/SKILL.md` (from SKILL.md.tmpl via gen-skill-docs)

- [ ] **Step 1: Run gen-skill-docs**

```bash
cd /Users/daedalium/Library/CloudStorage/Dropbox/GitHub/ostack && bun run gen:skill-docs
```

Expected: `startup/SKILL.md` is generated with all `{{PREAMBLE}}` placeholders resolved.

- [ ] **Step 2: Verify generation succeeded**

```bash
ls -la startup/SKILL.md
head -5 startup/SKILL.md
```

Expected: File exists, starts with frontmatter and generated header.

- [ ] **Step 3: Run validation tests**

```bash
bun run test:fast
```

Expected: All tests pass. The startup skill doesn't use `$B` commands (no browse), so skill-validation tests won't need to cover it.

- [ ] **Step 4: Commit generated file**

```bash
git add startup/SKILL.md
git commit -m "chore(startup): generate SKILL.md from template"
```

---

### Task 12: Register Skill in Root SKILL.md.tmpl and CLAUDE.md

**Files:**
- Modify: `SKILL.md.tmpl` (root) — add `/startup` to the skill suggestions list
- Modify: `CLAUDE.md` — add `/startup` to the available skills list and project structure

- [ ] **Step 1: Update root SKILL.md.tmpl description**

In the root `SKILL.md.tmpl`, find the `description:` field in the frontmatter. It lists skills by stage. Add `/startup` as the first stage (before brainstorm):

Find the line:
```
  Also suggest adjacent ostack skills by stage: brainstorm /office-hours; strategy
```

Replace with:
```
  Also suggest adjacent ostack skills by stage: bootstrap /startup; brainstorm /office-hours; strategy
```

- [ ] **Step 2: Update CLAUDE.md**

In `CLAUDE.md`, add `/startup` to the "Available ostack skills" list:

Find:
```
- `/office-hours` — brainstorming and idea validation
```

Add before it:
```
- `/startup` — bootstrap new project from empty folder
```

Also update the project structure section to include:
```
├── startup/         # /startup skill (project bootstrap from empty folder)
```

- [ ] **Step 3: Regenerate SKILL.md files**

```bash
bun run gen:skill-docs
```

- [ ] **Step 4: Run tests**

```bash
bun run test:fast
```

- [ ] **Step 5: Commit**

```bash
git add SKILL.md.tmpl SKILL.md CLAUDE.md startup/SKILL.md
git commit -m "feat(startup): register /startup skill in root SKILL.md and CLAUDE.md"
```

---

## Self-Review Checklist

1. **Spec coverage:** Every section of the design spec maps to a task:
   - Interview phase → Task 2
   - Architecture conversation → Tasks 3, 4
   - Service discovery → Task 5
   - All 10 provisioning steps → Tasks 6, 7, 8
   - All 7 file generators → Task 9
   - Ship + summary → Task 10
   - Gen-skill-docs integration → Task 11
   - Root skill registration → Task 12

2. **Placeholder scan:** No TBDs or TODOs in implementation steps. All code blocks are complete.

3. **Type consistency:** All file paths are consistent across tasks (e.g., `startup/routes/saas.md` is referenced the same way in Tasks 3 and 4). The `{slug}` placeholder is used consistently for GCP project IDs, Cloud Run services, and DNS zones.

4. **Missing from spec:** Nothing found — all spec requirements are covered by tasks.
