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
