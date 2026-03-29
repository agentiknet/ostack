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
