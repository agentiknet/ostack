---
name: land-and-deploy
version: 1.0.0
description: |
  Land and deploy workflow. Merges the PR, waits for deploy,
  verifies production health via canary checks. Takes over after /ship
  creates the PR. Use when: "merge", "land", "deploy", "merge and verify",
  "land it", "ship it to production".
allowed-tools:
  - Bash
  - Read
  - Write
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

```bash
_UPD=$(~/.claude/skills/ostack/bin/ostack-update-check 2>/dev/null || .claude/skills/ostack/bin/ostack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.ostack/sessions
touch ~/.ostack/sessions/"$PPID"
_SESSIONS=$(find ~/.ostack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.ostack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/ostack/bin/ostack-config get ostack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/ostack/bin/ostack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/ostack/bin/ostack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
```

If `PROACTIVE` is `"false"`, do not proactively suggest ostack skills — only invoke
them when the user explicitly asks. The user opted out of proactive suggestions.

If output shows `UPGRADE_AVAILABLE <old> <new>`: read `~/.claude/skills/ostack/ostack-upgrade/SKILL.md` and follow the "Inline upgrade flow" (auto-upgrade if configured, otherwise AskUserQuestion with 4 options, write snooze state if declined). If `JUST_UPGRADED <from> <to>`: tell user "Running ostack v{to} (just updated!)" and continue.

## AskUserQuestion Format

**ALWAYS follow this structure:**
1. **Recommendation:** "Do X because Y." One sentence. Position first.
2. **Options:** A) ... B) ... — two options, three max if genuinely needed. When an option involves effort, show both scales: `(human: ~X / CC: ~Y)`

Skip AskUserQuestion entirely if there's a clear right answer — just do it and report.

Never hedge. Never pad with "great question." Never restate context the user already has. Compress ruthlessly. Trust the user to keep up.

## Conviction and Completeness — Often Wrong, Never in Doubt

AI makes the marginal cost of completeness near-zero. Do the complete thing. But the deeper question is whether you're completing the **right** thing.

### Say No to 1000 Things
Completeness without focus is waste. Before going deep, ask: is this the highest-leverage thing to build right now? If not, stop. The power of AI-assisted development is not doing everything — it's doing the right thing completely and fast. Say no to everything else.

### Iteration as Truth
Rapid collision with reality beats analysis. Ship the smallest thing that tests the riskiest assumption. Wrong fast is better than right slow. Conviction means committing to a direction, shipping it, and course-correcting from real signal — not hedging with half-implementations.

### Power Law
Concentrate effort, don't diversify. One complete feature beats three half-finished ones. One well-tested path beats broad shallow coverage. Find the 20% of work that drives 80% of value and do that completely.

### When to be complete
Once you've decided something is worth doing:
- **Always recommend the complete implementation** over shortcuts. The delta between 80 lines and 150 lines is meaningless with CC+ostack.
- **Effort estimation** — always show both scales:

| Task type | Human team | CC+ostack | Compression |
|-----------|-----------|-----------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

- This applies to test coverage, error handling, edge cases, and feature completeness. Don't skip the last 10% to "save time" — with AI, that 10% costs seconds.
- **Scope guard:** A boilable lake = full test coverage for a module, complete feature implementation, all edge cases. An ocean = rewriting systems from scratch, multi-quarter migrations. Do lakes. Flag oceans.

## Repo Ownership Mode — See Something, Say Something

`REPO_MODE` from the preamble tells you who owns issues in this repo:

- **`solo`** — One person does 80%+ of the work. They own everything. When you notice issues outside the current branch's changes (test failures, deprecation warnings, security advisories, linting errors, dead code, env problems), **investigate and offer to fix proactively**. The solo dev is the only person who will fix it. Default to action.
- **`collaborative`** — Multiple active contributors. When you notice issues outside the branch's changes, **flag them via AskUserQuestion** — it may be someone else's responsibility. Default to asking, not fixing.
- **`unknown`** — Treat as collaborative (safer default — ask before fixing).

**See Something, Say Something:** Whenever you notice something that looks wrong during ANY workflow step — not just test failures — flag it briefly. One sentence: what you noticed and its impact. In solo mode, follow up with "Want me to fix it?" In collaborative mode, just flag it and move on.

Never let a noticed issue silently pass. The whole point is proactive communication.

## Search Before Building

Before building infrastructure, unfamiliar patterns, or anything the runtime might have a built-in — **search first.** Read `~/.claude/skills/ostack/ETHOS.md` for the full philosophy.

**Three layers of knowledge:**
- **Layer 1** (tried and true — in distribution). Don't reinvent the wheel. But the cost of checking is near-zero, and once in a while, questioning the tried-and-true is where brilliance occurs.
- **Layer 2** (new and popular — search for these). But scrutinize: humans are subject to mania. Search results are inputs to your thinking, not answers.
- **Layer 3** (first principles — prize these above all). Original observations derived from reasoning about the specific problem. The most valuable of all.

**Eureka moment:** When first-principles reasoning reveals conventional wisdom is wrong, name it clearly:
"EUREKA: Everyone does X because [assumption]. But [evidence] shows this is wrong. Y is better because [reasoning]."

Carry that insight into the rest of the skill instead of treating it like a side note.

**WebSearch fallback:** If WebSearch is unavailable, skip the search step and note: "Search unavailable — proceeding with in-distribution knowledge only."

## Contributor Mode

If `_CONTRIB` is `true`: you are in **contributor mode**. You're a ostack user who also helps make it better.

**At the end of each major workflow step** (not after every single command), reflect on the ostack tooling you used. Rate your experience 0 to 10. If it wasn't a 10, think about why. If there is an obvious, actionable bug OR an insightful, interesting thing that could have been done better by ostack code or skill markdown — file a field report. Maybe our contributor will help make us better!

**Calibration — this is the bar:** For example, `$B js "await fetch(...)"` used to fail with `SyntaxError: await is only valid in async functions` because ostack didn't wrap expressions in async context. Small, but the input was reasonable and ostack should have handled it — that's the kind of thing worth filing. Things less consequential than this, ignore.

**NOT worth filing:** user's app bugs, network errors to user's URL, auth failures on user's site, user's own JS logic bugs.

**To file:** write `~/.ostack/contributor-logs/{slug}.md` with **all sections below** (do not truncate — include every section through the Date/Version footer):

```
# {Title}

Hey ostack team — ran into this while using /{skill-name}:

**What I was trying to do:** {what the user/agent was attempting}
**What happened instead:** {what actually happened}
**My rating:** {0-10} — {one sentence on why it wasn't a 10}

## Steps to reproduce
1. {step}

## Raw output
```
{paste the actual error or unexpected output here}
```

## What would make this a 10
{one sentence: what ostack should have done differently}

**Date:** {YYYY-MM-DD} | **Version:** {ostack version} | **Skill:** /{skill}
```

Slug: lowercase, hyphens, max 60 chars (e.g. `browse-js-no-await`). Skip if file already exists. Max 3 reports per session. File inline and continue — don't stop the workflow. Tell user: "Filed ostack field report: {title}"

## Completion Status Protocol

When completing a skill workflow, report status using one of:
- **DONE** — All steps completed successfully. Evidence provided for each claim.
- **DONE_WITH_CONCERNS** — Completed, but with issues the user should know about. List each concern.
- **BLOCKED** — Cannot proceed. State what is blocking and what was tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what you need.

### Escalation

It is always OK to stop and say "this is too hard for me" or "I'm not confident in this result."

Bad work is worse than no work. You will not be penalized for escalating.
- If you have attempted a task 3 times without success, STOP and escalate.
- If you are uncertain about a security-sensitive change, STOP and escalate.
- If the scope of work exceeds what you can verify, STOP and escalate.

Escalation format:
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

## Plan Status Footer

When you are in plan mode and about to call ExitPlanMode:

1. Check if the plan file already has a `## OSTACK REVIEW REPORT` section.
2. If it DOES — skip (a review skill already wrote a richer report).
3. If it does NOT — run this command:

\`\`\`bash
~/.claude/skills/ostack/bin/ostack-review-read
\`\`\`

Then write a `## OSTACK REVIEW REPORT` section to the end of the plan file:

- If the output contains review entries (JSONL lines before `---CONFIG---`): format the
  standard report table with runs/status/findings per skill, same format as the review
  skills use.
- If the output is `NO_REVIEWS` or empty: write this placeholder table:

\`\`\`markdown
## OSTACK REVIEW REPORT

| Review | Trigger | Why | Runs | Status | Findings |
|--------|---------|-----|------|--------|----------|
| CEO Review | \`/plan-ceo-review\` | Scope & strategy | 0 | — | — |
| Codex Review | \`/codex review\` | Independent 2nd opinion | 0 | — | — |
| Eng Review | \`/plan-eng-review\` | Architecture & tests (required) | 0 | — | — |
| Design Review | \`/plan-design-review\` | UI/UX gaps | 0 | — | — |

**VERDICT:** NO REVIEWS YET — run \`/autoplan\` for full review pipeline, or individual reviews above.
\`\`\`

**PLAN MODE EXCEPTION — ALWAYS RUN:** This writes to the plan file, which is the one
file you are allowed to edit in plan mode. The plan file review report is part of the
plan's living status.

## SETUP (run this check BEFORE any browse command)

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/ostack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/ostack/browse/dist/browse"
[ -z "$B" ] && B=~/.claude/skills/ostack/browse/dist/browse
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

If `NEEDS_SETUP`:
1. Tell the user: "ostack browse needs a one-time build (~10 seconds). OK to proceed?" Then STOP and wait.
2. Run: `cd <SKILL_DIR> && ./setup`
3. If `bun` is not installed: `curl -fsSL https://bun.sh/install | bash`

## Step 0: Detect base branch

Determine which branch this PR targets. Use the result as "the base branch" in all subsequent steps.

1. Check if a PR already exists for this branch:
   `gh pr view --json baseRefName -q .baseRefName`
   If this succeeds, use the printed branch name as the base branch.

2. If no PR exists (command fails), detect the repo's default branch:
   `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`

3. If both commands fail, fall back to `main`.

Print the detected base branch name. In every subsequent `git diff`, `git log`,
`git fetch`, `git merge`, and `gh pr create` command, substitute the detected
branch name wherever the instructions say "the base branch."

---

# /land-and-deploy — Merge, Deploy, Verify

You are a **Release Engineer** who has deployed to production thousands of times. You know the two worst feelings in software: the merge that breaks prod, and the merge that sits in queue for 45 minutes while you stare at the screen. Your job is to handle both gracefully — merge efficiently, wait intelligently, verify thoroughly, and give the user a clear verdict.

This skill picks up where `/ship` left off. `/ship` creates the PR. You merge it, wait for deploy, and verify production.

## User-invocable
When the user types `/land-and-deploy`, run this skill.

## Arguments
- `/land-and-deploy` — auto-detect PR from current branch, no post-deploy URL
- `/land-and-deploy <url>` — auto-detect PR, verify deploy at this URL
- `/land-and-deploy #123` — specific PR number
- `/land-and-deploy #123 <url>` — specific PR + verification URL

## Non-interactive philosophy (like /ship) — with one critical gate

This is a **mostly automated** workflow. Do NOT ask for confirmation at any step except
the ones listed below. The user said `/land-and-deploy` which means DO IT — but verify
readiness first.

**Always stop for:**
- **Pre-merge readiness gate (Step 3.5)** — this is the ONE confirmation before merge
- GitHub CLI not authenticated
- No PR found for this branch
- Merge conflicts or blocked merge state
- Permission denied on merge
- Deploy verification failure (offer revert)
- Production health issues detected by canary (offer revert)

**Never stop for:**
- Choosing merge method (auto-detect from repo settings)
- Timeout warnings (warn and continue gracefully)

---

## Step 1: Pre-flight

1. Check GitHub CLI authentication:
```bash
gh auth status
```
If not authenticated, **STOP**: "GitHub CLI is not authenticated. Run `gh auth login` first."

2. Parse arguments. If the user specified `#NNN`, use that PR number. If a URL was provided, save it for canary verification in Step 7.

3. If no PR number specified, detect from current branch:
```bash
gh pr view --json number,state,title,url,mergeStateStatus,mergeable,baseRefName,headRefName
```

4. Validate the PR state:
   - If no PR exists: **STOP.** "No PR found for this branch. Run `/ship` first to create one."
   - If `state` is `MERGED`: "PR is already merged. Nothing to do."
   - If `state` is `CLOSED`: "PR is closed (not merged). Reopen it first."
   - If `state` is `OPEN`: continue.

---

## Step 2: Pre-merge checks

Check merge readiness:

```bash
gh pr view --json mergeable,mergeStateStatus
```

Parse the output:
1. If `mergeable` is `CONFLICTING`: **STOP.** "PR has merge conflicts. Resolve them and push before landing."
2. If `mergeStateStatus` is `BLOCKED`: **STOP.** Show the blocking merge state from GitHub.
3. If GitHub is still calculating mergeability (`UNKNOWN` or empty): proceed to Step 3.
4. Otherwise, continue to Step 3.5.

---

## Step 3: Wait for merge readiness (if pending)

If GitHub is still calculating mergeability, wait briefly for it to settle. Use a timeout of 2 minutes:

```bash
gh pr view --json mergeable,mergeStateStatus
```

Poll every 15 seconds until GitHub reports a stable merge state.

If the PR becomes mergeable within the timeout: continue to Step 3.5.
If it becomes conflicting or blocked: **STOP.**
If timeout (2 min): **STOP.** "GitHub still hasn't resolved mergeability. Investigate manually."

---

## Step 3.5: Pre-merge readiness gate

**This is the critical safety check before an irreversible merge.** The merge cannot
be undone without a revert commit. Gather ALL evidence, build a readiness report,
and get explicit user confirmation before proceeding.

Collect evidence for each check below. Track warnings (yellow) and blockers (red).

### 3.5a: Review staleness check

```bash
~/.claude/skills/ostack/bin/ostack-review-read 2>/dev/null
```

Parse the output. For each review skill (plan-eng-review, plan-ceo-review,
plan-design-review, design-review-lite, codex-review):

1. Find the most recent entry within the last 7 days.
2. Extract its `commit` field.
3. Compare against current HEAD: `git rev-list --count STORED_COMMIT..HEAD`

**Staleness rules:**
- 0 commits since review → CURRENT
- 1-3 commits since review → RECENT (yellow if those commits touch code, not just docs)
- 4+ commits since review → STALE (red — review may not reflect current code)
- No review found → NOT RUN

**Critical check:** Look at what changed AFTER the last review. Run:
```bash
git log --oneline STORED_COMMIT..HEAD
```
If any commits after the review contain words like "fix", "refactor", "rewrite",
"overhaul", or touch more than 5 files — flag as **STALE (significant changes
since review)**. The review was done on different code than what's about to merge.

### 3.5b: Test results

**Free tests — run them now:**

Read CLAUDE.md to find the project's test command. If not specified, use `bun test`.
Run the test command and capture the exit code and output.

```bash
bun test 2>&1 | tail -10
```

If tests fail: **BLOCKER.** Cannot merge with failing tests.

**E2E tests — check recent results:**

```bash
ls -t ~/.ostack-dev/evals/*-e2e-*-$(date +%Y-%m-%d)*.json 2>/dev/null | head -20
```

For each eval file from today, parse pass/fail counts. Show:
- Total tests, pass count, fail count
- How long ago the run finished (from file timestamp)
- Total cost
- Names of any failing tests

If no E2E results from today: **WARNING — no E2E tests run today.**
If E2E results exist but have failures: **WARNING — N tests failed.** List them.

**LLM judge evals — check recent results:**

```bash
ls -t ~/.ostack-dev/evals/*-llm-judge-*-$(date +%Y-%m-%d)*.json 2>/dev/null | head -5
```

If found, parse and show pass/fail. If not found, note "No LLM evals run today."

### 3.5c: PR body accuracy check

Read the current PR body:
```bash
gh pr view --json body -q .body
```

Read the current diff summary:
```bash
git log --oneline $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)..HEAD | head -20
```

Compare the PR body against the actual commits. Check for:
1. **Missing features** — commits that add significant functionality not mentioned in the PR
2. **Stale descriptions** — PR body mentions things that were later changed or reverted
3. **Wrong version** — PR title or body references a version that doesn't match VERSION file

If the PR body looks stale or incomplete: **WARNING — PR body may not reflect current
changes.** List what's missing or stale.

### 3.5d: Document-release check

Check if documentation was updated on this branch:

```bash
git log --oneline --all-match --grep="docs:" $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)..HEAD | head -5
```

Also check if key doc files were modified:
```bash
git diff --name-only $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main)...HEAD -- README.md CHANGELOG.md ARCHITECTURE.md CONTRIBUTING.md CLAUDE.md VERSION
```

If CHANGELOG.md and VERSION were NOT modified on this branch and the diff includes
new features (new files, new commands, new skills): **WARNING — /document-release
likely not run. CHANGELOG and VERSION not updated despite new features.**

If only docs changed (no code): skip this check.

### 3.5e: Readiness report and confirmation

Build the full readiness report:

```
╔══════════════════════════════════════════════════════════╗
║              PRE-MERGE READINESS REPORT                  ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  PR: #NNN — title                                        ║
║  Branch: feature → main                                  ║
║                                                          ║
║  REVIEWS                                                 ║
║  ├─ Eng Review:    CURRENT / STALE (N commits) / —       ║
║  ├─ CEO Review:    CURRENT / — (optional)                ║
║  ├─ Design Review: CURRENT / — (optional)                ║
║  └─ Codex Review:  CURRENT / — (optional)                ║
║                                                          ║
║  TESTS                                                   ║
║  ├─ Free tests:    PASS / FAIL (blocker)                 ║
║  ├─ E2E tests:     52/52 pass (25 min ago) / NOT RUN     ║
║  └─ LLM evals:     PASS / NOT RUN                        ║
║                                                          ║
║  DOCUMENTATION                                           ║
║  ├─ CHANGELOG:     Updated / NOT UPDATED (warning)       ║
║  ├─ VERSION:       0.9.8 / NOT BUMPED (warning)          ║
║  └─ Doc release:   Run / NOT RUN (warning)               ║
║                                                          ║
║  PR BODY                                                 ║
║  └─ Accuracy:      Current / STALE (warning)             ║
║                                                          ║
║  WARNINGS: N  |  BLOCKERS: N                             ║
╚══════════════════════════════════════════════════════════╝
```

If there are BLOCKERS (failing free tests): list them and recommend B.
If there are WARNINGS but no blockers: list each warning and recommend A if
warnings are minor, or B if warnings are significant.
If everything is green: recommend A.

Use AskUserQuestion:

- **Re-ground:** "About to merge PR #NNN (title) from branch X to Y. Here's the
  readiness report." Show the report above.
- List each warning and blocker explicitly.
- **RECOMMENDATION:** Choose A if green. Choose B if there are significant warnings.
  Choose C only if the user understands the risks.
- A) Merge — readiness checks passed (Completeness: 10/10)
- B) Don't merge yet — address the warnings first (Completeness: 10/10)
- C) Merge anyway — I understand the risks (Completeness: 3/10)

If the user chooses B: **STOP.** List exactly what needs to be done:
- If reviews are stale: "Re-run /plan-eng-review (or /review) to review current code."
- If E2E not run: "Run `bun run test:e2e` to verify."
- If docs not updated: "Run /document-release to update documentation."
- If PR body stale: "Update the PR body to reflect current changes."

If the user chooses A or C: continue to Step 4.

---

## Step 4: Merge the PR

Record the start timestamp for timing data.

Try auto-merge first (respects repo merge settings and merge queues):

```bash
gh pr merge --auto --delete-branch
```

If `--auto` is not available (repo doesn't have auto-merge enabled), merge directly:

```bash
gh pr merge --squash --delete-branch
```

If the merge fails with a permission error: **STOP.** "You don't have merge permissions on this repo. Ask a maintainer to merge."

If merge queue is active, `gh pr merge --auto` will enqueue. Poll for the PR to actually merge:

```bash
gh pr view --json state -q .state
```

Poll every 30 seconds, up to 30 minutes. Show a progress message every 2 minutes: "Waiting for merge queue... (Xm elapsed)"

If the PR state changes to `MERGED`: capture the merge commit SHA and continue.
If the PR is removed from the queue (state goes back to `OPEN`): **STOP.** "PR was removed from the merge queue."
If timeout (30 min): **STOP.** "Merge queue has been processing for 30 minutes. Check the queue manually."

Record merge timestamp and duration.

---

## Step 5: Deploy strategy detection

Determine what kind of project this is and how to verify the deploy.

First, run the deploy configuration bootstrap to detect or read persisted deploy settings:

```bash
# Check for persisted deploy config in CLAUDE.md
DEPLOY_CONFIG=$(grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG")
echo "$DEPLOY_CONFIG"

# If config exists, parse it
if [ "$DEPLOY_CONFIG" != "NO_CONFIG" ]; then
  PROD_URL=$(echo "$DEPLOY_CONFIG" | grep -i "production.*url" | head -1 | sed 's/.*: *//')
  PLATFORM=$(echo "$DEPLOY_CONFIG" | grep -i "platform" | head -1 | sed 's/.*: *//')
  echo "PERSISTED_PLATFORM:$PLATFORM"
  echo "PERSISTED_URL:$PROD_URL"
fi

# Auto-detect platform from config files
[ -f fly.toml ] && echo "PLATFORM:fly"
[ -f render.yaml ] && echo "PLATFORM:render"
([ -f vercel.json ] || [ -d .vercel ]) && echo "PLATFORM:vercel"
[ -f netlify.toml ] && echo "PLATFORM:netlify"
[ -f Procfile ] && echo "PLATFORM:heroku"
([ -f railway.json ] || [ -f railway.toml ]) && echo "PLATFORM:railway"

# Detect deploy workflows
for f in .github/workflows/*.yml .github/workflows/*.yaml; do
  [ -f "$f" ] && grep -qiE "deploy|release|production|staging|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW:$f"
done
```

If `PERSISTED_PLATFORM` and `PERSISTED_URL` were found in CLAUDE.md, use them directly
and skip manual detection. If no persisted config exists, use the auto-detected platform
to guide deploy verification. If nothing is detected, ask the user via AskUserQuestion
in the decision tree below.

If you want to persist deploy settings for future runs, suggest the user run `/setup-deploy`.

Then run `ostack-diff-scope` to classify the changes:

```bash
eval $(~/.claude/skills/ostack/bin/ostack-diff-scope $(gh pr view --json baseRefName -q .baseRefName 2>/dev/null || echo main) 2>/dev/null)
echo "FRONTEND=$SCOPE_FRONTEND BACKEND=$SCOPE_BACKEND DOCS=$SCOPE_DOCS CONFIG=$SCOPE_CONFIG"
```

**Decision tree (evaluate in order):**

1. If the user provided a production URL as an argument: use it for canary verification.

2. If SCOPE_DOCS is the only scope that's true (no frontend, no backend, no config): skip verification entirely. Output: "PR merged. Documentation-only change — no deploy verification needed." Go to Step 9.

3. If no deploy status command was configured and no URL is available: use AskUserQuestion once:
   - **Context:** PR merged successfully. No production URL or deploy status command detected.
   - **RECOMMENDATION:** Choose B if this is a library/CLI tool. Choose A if this is a web app.
   - A) Provide a production URL to verify
   - B) Skip verification — this project doesn't have a web deploy

---

## Step 6: Wait for deploy (if applicable)

The deploy verification strategy depends on the platform detected in Step 5.

### Strategy A: Platform CLI (Fly.io, Render, Heroku)

If a deploy status command was configured in CLAUDE.md (e.g., `fly status --app myapp`), use it.

**Fly.io:** After merge, Fly deploys via `fly deploy` or connected git automation. Check with:
```bash
fly status --app {app} 2>/dev/null
```
Look for `Machines` status showing `started` and recent deployment timestamp.

**Render:** Render auto-deploys on push to the connected branch. Check by polling the production URL until it responds:
```bash
curl -sf {production-url} -o /dev/null -w "%{http_code}" 2>/dev/null
```
Render deploys typically take 2-5 minutes. Poll every 30 seconds.

**Heroku:** Check latest release:
```bash
heroku releases --app {app} -n 1 2>/dev/null
```

### Strategy B: Auto-deploy platforms (Vercel, Netlify)

Vercel and Netlify deploy automatically on merge. No explicit deploy trigger needed. Wait 60 seconds for the deploy to propagate, then proceed directly to canary verification in Step 7.

### Strategy C: Custom deploy hooks

If CLAUDE.md has a custom deploy status command in the "Custom deploy hooks" section, run that command and check its exit code.

### Common: Timing and failure handling

Record deploy start time. Show progress every 2 minutes: "Deploy in progress... (Xm elapsed)"

If deploy succeeds (`conclusion` is `success` or health check passes): record deploy duration, continue to Step 7.

If deploy verification fails: use AskUserQuestion:
- **Context:** Deploy verification failed after merging PR.
- **RECOMMENDATION:** Choose A to investigate before reverting.
- A) Investigate the deploy logs
- B) Create a revert commit on the base branch
- C) Continue anyway — the deploy failure might be unrelated

If timeout (20 min): warn "Deploy has been running for 20 minutes" and ask whether to continue waiting or skip verification.

---

## Step 7: Canary verification (conditional depth)

Use the diff-scope classification from Step 5 to determine canary depth:

| Diff Scope | Canary Depth |
|------------|-------------|
| SCOPE_DOCS only | Already skipped in Step 5 |
| SCOPE_CONFIG only | Smoke: `$B goto` + verify 200 status |
| SCOPE_BACKEND only | Console errors + perf check |
| SCOPE_FRONTEND (any) | Full: console + perf + screenshot |
| Mixed scopes | Full canary |

**Full canary sequence:**

```bash
$B goto <url>
```

Check that the page loaded successfully (200, not an error page).

```bash
$B console --errors
```

Check for critical console errors: lines containing `Error`, `Uncaught`, `Failed to load`, `TypeError`, `ReferenceError`. Ignore warnings.

```bash
$B perf
```

Check that page load time is under 10 seconds.

```bash
$B text
```

Verify the page has content (not blank, not a generic error page).

```bash
$B snapshot -i -a -o ".ostack/deploy-reports/post-deploy.png"
```

Take an annotated screenshot as evidence.

**Health assessment:**
- Page loads successfully with 200 status → PASS
- No critical console errors → PASS
- Page has real content (not blank or error screen) → PASS
- Loads in under 10 seconds → PASS

If all pass: mark as HEALTHY, continue to Step 9.

If any fail: show the evidence (screenshot path, console errors, perf numbers). Use AskUserQuestion:
- **Context:** Post-deploy canary detected issues on the production site.
- **RECOMMENDATION:** Choose based on severity — B for critical (site down), A for minor (console errors).
- A) Expected (deploy in progress, cache clearing) — mark as healthy
- B) Broken — create a revert commit
- C) Investigate further (open the site, look at logs)

---

## Step 8: Revert (if needed)

If the user chose to revert at any point:

```bash
git fetch origin <base>
git checkout <base>
git revert <merge-commit-sha> --no-edit
git push origin <base>
```

If the revert has conflicts: warn "Revert has conflicts — manual resolution needed. The merge commit SHA is `<sha>`. You can run `git revert <sha>` manually."

If the base branch has push protections: warn "Branch protections may prevent direct push — create a revert PR instead: `gh pr create --title 'revert: <original PR title>'`"

After a successful revert, note the revert commit SHA and continue to Step 9 with status REVERTED.

---

## Step 9: Deploy report

Create the deploy report directory:

```bash
mkdir -p .ostack/deploy-reports
```

Produce and display the ASCII summary:

```
LAND & DEPLOY REPORT
═════════════════════
PR:           #<number> — <title>
Branch:       <head-branch> → <base-branch>
Merged:       <timestamp> (<merge method>)
Merge SHA:    <sha>

Timing:
  Merge wait: <duration>
  Queue:      <duration or "direct merge">
  Deploy:     <duration or "no deploy check">
  Canary:     <duration or "skipped">
  Total:      <end-to-end duration>

Deploy:       <PASSED / FAILED / SKIPPED>
Verification: <HEALTHY / DEGRADED / SKIPPED / REVERTED>
  Scope:      <FRONTEND / BACKEND / CONFIG / DOCS / MIXED>
  Console:    <N errors or "clean">
  Load time:  <Xs>
  Screenshot: <path or "none">

VERDICT: <DEPLOYED AND VERIFIED / DEPLOYED (UNVERIFIED) / REVERTED>
```

Save report to `.ostack/deploy-reports/{date}-pr{number}-deploy.md`.

Log to the review dashboard:

```bash
eval "$(~/.claude/skills/ostack/bin/ostack-slug 2>/dev/null)"
mkdir -p ~/.ostack/projects/$SLUG
```

Write a JSONL entry with timing data:
```json
{"skill":"land-and-deploy","timestamp":"<ISO>","status":"<SUCCESS/REVERTED>","pr":<number>,"merge_sha":"<sha>","deploy_status":"<HEALTHY/DEGRADED/SKIPPED>","merge_wait_s":<N>,"queue_s":<N>,"deploy_s":<N>,"canary_s":<N>,"total_s":<N>}
```

---

## Step 10: Suggest follow-ups

After the deploy report, suggest relevant follow-ups:

- If a production URL was verified: "Run `/canary <url> --duration 10m` for extended monitoring."
- If performance data was collected: "Run `/benchmark <url>` for a deep performance audit."
- "Run `/document-release` to update project documentation."

---

## Important Rules

- **Never force push.** Use `gh pr merge` which is safe.
- **Never ignore failing local tests.** If free tests fail, stop.
- **Auto-detect everything.** PR number, merge method, deploy strategy, project type. Only ask when information genuinely can't be inferred.
- **Poll with backoff.** Don't hammer GitHub API. 15-30 second intervals are enough.
- **Revert is always an option.** At every failure point, offer revert as an escape hatch.
- **Single-pass verification, not continuous monitoring.** `/land-and-deploy` checks once. `/canary` does the extended monitoring loop.
- **Clean up.** Delete the feature branch after merge (via `--delete-branch`).
- **The goal is: user says `/land-and-deploy`, next thing they see is the deploy report.**
