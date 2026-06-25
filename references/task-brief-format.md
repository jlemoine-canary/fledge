# Task brief format — `TASK-BRIEF.md`

The deterministic, minimal brief handed to the `fledge-implementer` subagent. This is the
single source of truth for how the implementer's hand-off is assembled.
`skills/fledge-implement/SKILL.md` (step 2b) produces it; `agents/fledge-implementer.md`
consumes it (its `Inputs` are this file).

The point is **reproducibility and minimality**: the brief is assembled by a fixed recipe
from artifacts that already exist, so two runs produce the same fields and the orchestrator
never re-includes whole artifact bodies into its own context. The implementer has `Read` —
it fetches what it is pointed at.

## The determinism rule

Every field is **mechanically derived** from `PLAN.md`, `TESTS.md`, and
`REVIEW-PLAN-final.md`. The compiler transcribes references and extracts the review residue —
it does not paraphrase the plan, paste artifact bodies, invent test commands, or vary which
obligations it lists run to run. Same inputs → same brief.

**References, not contents.** The brief passes *paths*; it never inlines the body of
`PLAN.md`, the review, or the tests. (Context-budget protocol.) The one exception is the test
command, which is lifted **verbatim** from `TESTS.md`.

## Source of each field

| Field | Derived from |
|---|---|
| Phase id + directory | the phase being implemented |
| Plan / tests / review paths | the phase directory (paths only) |
| Surface | `PLAN.md` `## Surface` |
| File plan | `PLAN.md` `## File plan` (transcribed, not re-described) |
| Failing tests + run command | `TESTS.md` (verbatim — including the exact command) |
| Required fixes | `REVIEW-PLAN-final.md` blocking findings, if any remain |
| Deferred nits | `REVIEW-PLAN-final.md` non-blocking findings, marked do-not-act |
| SoT id + manifest path | `.fledge/SOURCES.md` (the `sot: true` entry) |
| Project `CLAUDE.md` path | the standards file for this surface (e.g. `backend/CLAUDE.md`) |
| Iteration cap | fixed: 5 per test, then `BLOCKED-*.md` |

## Output structure (fixed sections, fixed order)

Write `.fledge/phases/<id>/TASK-BRIEF.md` with exactly these sections:

```markdown
# Task brief — Phase <id>

Implement this phase via strict TDD. Read the referenced files yourself (you have Read);
they are not pasted here.

- **Phase directory:** `.fledge/phases/<id>/`
- **Plan:** `.fledge/phases/<id>/PLAN.md`
- **Tests ledger:** `.fledge/phases/<id>/TESTS.md`
- **Plan review (final):** `.fledge/phases/<id>/REVIEW-PLAN-final.md`
- **Source manifest:** `.fledge/SOURCES.md` — SoT: `<id>` (authoritative on ambiguity)
- **Standards:** `<path to CLAUDE.md>` (wins over the plan on conflict)
- **Surface:** <surface>

## Failing tests (success criterion — confirmed RED)
<verbatim list from TESTS.md>

## Test command
<verbatim from TESTS.md>

## File plan (stay within this scope)
<transcribed from PLAN.md ## File plan>

## Required fixes (blocking)
<blocking findings from REVIEW-PLAN-final.md; "None — verdict PASS." if clean>

## Deferred — do NOT act on
<non-blocking nits from REVIEW-PLAN-final.md, each marked deferred; "None." if clean>

## Iteration cap
5 attempts per test. On the 6th failure, STOP and write `BLOCKED-<test>.md`.
Escalations: `DEVIATION.md` (plan contradicts SoT), `TEST-DEFECT.md` (test is wrong — never edit a test to pass).

## On completion
Write `IMPLEMENTATION.md` (use `references/templates/implementation.md`), every cleanup item
ticked truthfully. Leave changes uncommitted — the orchestrator runs post-implementation
sanity (full suite + lint/typecheck) and owns the commit.
```

Do **not** add sections beyond these, and do **not** drop the `On completion` block — the
implementer persona writes `IMPLEMENTATION.md` and the orchestrator owns the commit; a brief
that omits or contradicts this breaks the pipeline contract.

## What this brief does NOT contain
- No pasted artifact bodies (plan text, review text, test source) — paths only.
- No invented or paraphrased test command — lift it verbatim from `TESTS.md`.
- No commit/push authority for the implementer — the orchestrator commits.
- No re-litigation of deferred nits as blockers, and no silent dropping of them.
