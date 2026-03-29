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
