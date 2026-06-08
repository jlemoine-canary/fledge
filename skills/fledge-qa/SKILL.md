---
name: fledge-qa
description: End-to-end QA with Playwright MCP. Derives scenarios from the source-of-truth, runs them in a real browser, and iterates with the implementer on failures. Up to 3 rounds. Use when the user says "QA this phase", "run Playwright", or after /fledge:fledge-review code passes.
version: 0.1.0
---

# fledge-qa

Final stage. Browser-level QA via the Playwright MCP. Spawns `fledge-qa-engineer`. Iterates with `/fledge:fledge-implement` on any failures.

This skill returns a verdict — it does NOT checkpoint. The orchestrator (`/fledge:fledge`) decides when to checkpoint.

## Preconditions

- `REVIEW-CODE-final.md` verdict = PASS
- `/fledge:fledge-auth` confirmed Playwright MCP is available
- Project has a working local dev environment (as described in `CLAUDE.md`)

## Arguments

- **Phase id** (defaults to most recent phase with a PASS code review)
- **`--app`** — specify which SPA / service if the phase touches multiple (e.g. `--app=check-in`)

## Process

### 1. Confirm dev server availability

Per `CLAUDE.md`:
- Frontend: `http://localhost:8080`
- Backend: `http://localhost:8000` or the service's port
- Other apps as specified

Probe with `curl` via Bash or a cheap Playwright navigation. If not running:
- For example: `cd backend && make up` (or the variant from `CLAUDE.md`)
- Ask the user before starting anything heavy

### 2. Spawn `fledge-qa-engineer`

Pass:
- Phase dir, `PLAN.md`, `IMPLEMENTATION.md`, `REVIEW-CODE-final.md` paths
- SoT snapshot path (refresh first per `references/sot-snapshot.md`)
- Project `CLAUDE.md`
- App under test (from arg or inferred)
- Dev server URL(s)
- Output template: `references/templates/qa.md`

The engineer writes `QA.md` (scenarios + statuses, matching the template) and optionally `QA-FINDINGS.md` (if anything failed, using the format in `references/templates/review.md`).

### 3. Handle failures

If `QA-FINDINGS.md` has consequential findings:
1. Route back to `/fledge:fledge-implement` with the findings as required fixes
2. On re-implementation, re-spawn `/fledge:fledge-review code` (the review cycle must re-approve the fix)
3. Then re-spawn `fledge-qa-engineer`

Max 3 QA rounds. On round 3 still failing, return `ESCALATE` to the orchestrator.

## Return value

```yaml
verdict: PASS | ESCALATE
rounds_used: <N>
qa_md_path: <path>
findings_path: <path or null>
scenarios_pass: <N>
scenarios_fail: <N>
```

## Playwright guidance

The `fledge-qa-engineer` subagent has the browser tools. The skill itself does not drive the browser — it just orchestrates.

If the project already has a Playwright test suite (check for `playwright.config.ts` or equivalent), the engineer adds to it. If not, the engineer writes standalone tests in `tests/e2e/fledge-<phase-id>/` with a minimal `playwright.config.ts`.

## Context-budget management

Browser snapshots and screenshots can be token-heavy. The engineer should:
- Take a snapshot only on failure
- Use ARIA snapshots over screenshots when possible (smaller, more stable)
- Summarize passing scenarios in a single table, not per-scenario narrative

## What this skill does NOT do
- Run unit tests (those ran in `/fledge:fledge-test` / `/fledge:fledge-implement`)
- Test production or staging — **local only**
- Fix the code — it routes failures back to `/fledge:fledge-implement`
- Checkpoint the user — the orchestrator does that based on the verdict

## Tools needed
- `Read`, `Write`, `Bash` (for dev server probes, SoT snapshot)
- `Agent` (spawn `fledge-qa-engineer`, and re-spawn `/fledge:fledge-implement` via orchestrator on failure)

## Related
- Subagent: `fledge-qa-engineer`
- References: `severity-rubric.md`, `sot-snapshot.md`, `templates/qa.md`, `templates/review.md`
- Back-step on failure: `/fledge:fledge-implement` then `/fledge:fledge-review code`
