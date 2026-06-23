---
name: fledge-qa
description: Surface-aware end-to-end QA. Classifies the phase as frontend/backend/full-stack/library-internal, then runs the matching QA — Playwright browser flows for UI, API + side-effect checks against the local service for backend, or a justified skip for pure library/CLI work. Derives scenarios from the source-of-truth and iterates with the implementer on failures. Up to 3 rounds. Use when the user says "QA this phase", "run QA", or after /fledge:fledge-review code passes.
---

# fledge-qa

Final stage. **Surface-aware** QA: not every phase is a browser phase. This skill classifies
the phase's surface and routes to the right QA — see `references/qa-by-surface.md` for the
taxonomy and the local-only guardrail. It spawns `fledge-qa-engineer` (browser) and/or
`fledge-qa-engineer-backend` (API/side-effects), and iterates with `/fledge:fledge-implement`
on any failures.

This skill returns a verdict — it does NOT checkpoint. The orchestrator (`/fledge:fledge`) decides when to checkpoint.

## Preconditions

- `REVIEW-CODE-final.md` verdict = PASS
- Project has a working **local** dev environment (as described in `CLAUDE.md`)
- **For `frontend`/`full-stack` phases only:** `/fledge:fledge-auth` confirmed the Playwright MCP is available. Backend-only and library-internal phases do **not** require Playwright.

## Arguments

- **Phase id** (defaults to most recent phase with a PASS code review)
- **`--app`** — specify which SPA / service if the phase touches multiple (e.g. `--app=check-in`)
- **`--surface=<frontend|backend|full-stack|library-internal>`** — override the classification when the plan's `## Surface` field is missing or wrong

## Process

### 1. Classify the phase surface

Determine the surface per `references/qa-by-surface.md`:
1. Read the `## Surface` field in the phase's `PLAN.md` (authoritative).
2. If absent, infer from the `File plan` table and the git diff (frontend roots → `frontend`, backend roots → `backend`, both → `full-stack`, only internal/CLI/migrations → `library-internal`).
3. A `--surface=` arg overrides both.

The surface decides which agent(s) run and whether Playwright is needed at all.

### 2. Confirm the relevant dev server(s)

Only probe what the surface needs (per `CLAUDE.md`):
- `frontend` → frontend dev server (e.g. `http://localhost:8080`)
- `backend` → the local service (e.g. `http://localhost:8000`, or the service's port)
- `full-stack` → both
- `library-internal` → no server; skip this step

Probe with `curl` via Bash (or a cheap Playwright navigation for the frontend). If not running:
- For example: `cd backend && make up` (or the variant from `CLAUDE.md`)
- Ask the user before starting anything heavy
- **Local only** — never point QA at staging or production.

### 3. Spawn the QA agent(s) for the surface

- `frontend` → spawn `fledge-qa-engineer` (Playwright)
- `backend` → spawn `fledge-qa-engineer-backend` (API + side-effects)
- `full-stack` → spawn both (backend first to confirm the contract, then browser to confirm the UI consumes it)
- `library-internal` → spawn `fledge-qa-engineer-backend` with a `library-internal` brief: it records a justified skip in `QA.md` (or runs a CLI smoke), citing the test-stage coverage. Do not invent browser scenarios.

Pass to whichever agent(s) run:
- Phase dir, `PLAN.md` (incl. its `## Surface`), `IMPLEMENTATION.md`, `REVIEW-CODE-final.md` paths
- SoT snapshot path (refresh first per `references/sot-snapshot.md`)
- Project `CLAUDE.md`
- App/service under test (from arg or inferred)
- Dev server URL(s) for the surface
- The playbook reference: `references/qa-by-surface.md`
- Output template: `references/templates/qa.md`

Each engineer writes `QA.md` (scenarios + statuses, matching the template, with the `Surface:` line set) and optionally `QA-FINDINGS.md` (if anything failed, using the format in `references/templates/review.md`). For full-stack, the two agents append to a single `QA.md` (one Scenarios table, surface noted per row) rather than overwriting each other.

### 4. Handle failures

If `QA-FINDINGS.md` has consequential findings:
1. Route back to `/fledge:fledge-implement` with the findings as required fixes
2. On re-implementation, re-spawn `/fledge:fledge-review code` (the review cycle must re-approve the fix)
3. Then re-spawn the QA agent for the surface (`fledge-qa-engineer` and/or `fledge-qa-engineer-backend`) — only the one(s) whose scenarios failed need re-running.

Max 3 QA rounds. On round 3 still failing, return `ESCALATE` to the orchestrator.

## Return value

```yaml
verdict: PASS | ESCALATE
surface: frontend | backend | full-stack | library-internal
rounds_used: <N>
qa_md_path: <path>
findings_path: <path or null>
scenarios_pass: <N>
scenarios_fail: <N>
scenarios_skipped: <N>   # e.g. library-internal justified skips
```

## QA scaffolding guidance

The subagents do the driving; the skill only orchestrates.

- **Frontend** (`fledge-qa-engineer`): if the project already has a Playwright suite (check for `playwright.config.ts` or equivalent), add to it; otherwise write standalone tests in `tests/e2e/fledge-<phase-id>/` with a minimal config.
- **Backend** (`fledge-qa-engineer-backend`): prefer extending the project's existing API/integration harness; otherwise drive the local service with `httpx`/`curl`. No browser, no Playwright config.

## Context-budget management

Browser snapshots and screenshots can be token-heavy. The frontend engineer should:
- Take a snapshot only on failure
- Use ARIA snapshots over screenshots when possible (smaller, more stable)
- Summarize passing scenarios in a single table, not per-scenario narrative

The backend engineer should summarize passing scenarios in one table and include full request/response detail only in findings.

## What this skill does NOT do
- Run unit tests (those ran in `/fledge:fledge-test` / `/fledge:fledge-implement`)
- Force browser QA onto a backend or library phase — it classifies the surface first
- Test production or staging — **local only**
- Fix the code — it routes failures back to `/fledge:fledge-implement`
- Checkpoint the user — the orchestrator does that based on the verdict

## Tools needed
- `Read`, `Write`, `Bash` (for dev server probes, SoT snapshot)
- `Agent` (spawn `fledge-qa-engineer` and/or `fledge-qa-engineer-backend`, and re-spawn `/fledge:fledge-implement` via orchestrator on failure)

## Related
- Subagents: `fledge-qa-engineer` (frontend/browser), `fledge-qa-engineer-backend` (API/side-effects)
- References: `qa-by-surface.md`, `severity-rubric.md`, `sot-snapshot.md`, `templates/qa.md`, `templates/review.md`
- Back-step on failure: `/fledge:fledge-implement` then `/fledge:fledge-review code`
