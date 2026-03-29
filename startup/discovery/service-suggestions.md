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
