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
