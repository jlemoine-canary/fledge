# Scenario: fledge-implement / task-brief-minimal

> Fixture for `/fledge:fledge-eval`. Exercises the deterministic task-brief handoff:
> the implementer subagent should receive a generated, minimal brief assembled by a fixed
> recipe — not an ad-hoc prompt that re-includes whole artifacts or varies run to run.

## Skill under test
`fledge-implement` (step b — spawn the implementer) deferring to
`references/task-brief-format.md`. Given a reviewed phase, the brief handed to the
implementer must be a fixed, minimal set of references, assembled the same way every run.

## Task prompt
You are the orchestrator about to spawn the implementer subagent for the fledge phase
`02-booking-pricing` at `.fledge/phases/02-booking-pricing/`. The plan passed review.
Prepare the brief you will hand the implementer. State exactly what you put in it and show it.

These artifacts exist in the phase directory (treat the excerpts as their real contents):

`PLAN.md` (excerpt):
```
## Surface
backend

## File plan
| File | Change |
|---|---|
| backend/bookings/pricing.py | add compute_total: discount before tax |
| backend/bookings/models.py  | add loyalty_discount_pct field |

## Test plan (TDD)
- R1 → test_total_applies_loyalty_discount_before_tax
```

`TESTS.md` (excerpt):
```
Red state confirmed. Failing tests:
- backend/bookings/tests/test_pricing.py::test_total_applies_loyalty_discount_before_tax
Run with: cd backend && pytest bookings/tests/test_pricing.py -x
```

`REVIEW-PLAN-final.md` (excerpt):
```
Verdict: PASS
Non-blocking nits:
- N1 (minor): consider Decimal over float for money — defer, not blocking.
```

`.fledge/SOURCES.md` marks `S1 — STAY-2122 (linear-issue)` as `sot: true`.
Project standards live in `backend/CLAUDE.md`.

## Setup / context
- Both arms get exactly the excerpts above.
- The implementer persona has `Read` and will fetch any file it is given a path to — it does
  not need artifact bodies pasted into its prompt (context-budget protocol: references, not
  contents).

## Pass criteria (observable)
1. The brief passes **references (paths)**, not pasted artifact bodies — e.g. it gives the
   `PLAN.md` path, not the plan's full text inline.
2. The brief contains a **fixed minimal set**: phase dir; paths to PLAN.md / TESTS.md /
   REVIEW-PLAN-final.md; the failing-test file list **and the exact test command**
   (`cd backend && pytest ...`); the SoT id (`STAY-2122`) + source manifest path; the
   project `CLAUDE.md` path; the iteration cap.
3. It **extracts the actionable review residue** — the non-blocking nit N1 is carried as a
   deferred note, not silently dropped and not promoted to a blocker.
4. A second independent run produces a brief with the **same fields**, not a differently
   shaped prose prompt.

## Known failure modes (what the WITHOUT arm is expected to do)
- Pastes whole artifact bodies (PLAN.md, the review) into the prompt — blowing the context
  budget the protocol exists to protect.
- Omits the exact test command, or invents one that differs from `TESTS.md`.
- Drops the review's non-blocking nits entirely, or re-litigates them as blockers.
- Produces a free-form prose hand-off whose shape varies run to run.

## Notes
- Subagent output is stochastic. Run each arm ≥2× and compare structure. The WITHOUT arm's
  tell is over-inclusion (pasting bodies) and run-to-run shape drift; the WITH arm should be
  a stable, minimal, path-based brief.
