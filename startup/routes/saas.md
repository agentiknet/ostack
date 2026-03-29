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
