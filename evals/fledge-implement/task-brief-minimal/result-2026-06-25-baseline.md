# Eval result: fledge-implement / task-brief-minimal — 2026-06-25-baseline

> Produced per `references/skill-eval-protocol.md`. RED→GREEN delta for the
> `references/task-brief-format.md` recipe (issue #3).

- **Scenario:** `evals/fledge-implement/task-brief-minimal/scenario.md`
- **Mode:** delta
- **Runs per arm:** WITHOUT ×2, WITH ×1

## Scorecard
| Pass criterion | WITHOUT arm | WITH arm |
|---|---|---|
| 1. Passes references (paths), not pasted bodies | pass (both) | pass |
| 2. Fixed minimal field set (incl. exact test command) | partial | pass |
| 3. Review residue carried (N1 deferred, not dropped/promoted) | pass (both) | pass |
| 4. Second run has the same shape | fail | pass (matches recipe) |

## Delta
- **Fixed by the recipe:** criteria 2 and 4. The context-budget protocol is already
  documented, so both WITHOUT runs passed paths and carried N1 — but the *shape and the set
  of obligations* drifted run to run. The recipe pins them.
- **Not fixed:** nothing material.
- **New rationalizations observed:** none in the WITH arm.

## Transcript excerpts
### WITHOUT arm (divergence)
- Run 1: long brief with a post-impl "After the test passes" sanity block, withheld commit
  authority, and an instruction to write `IMPLEMENTATION.md` from the template.
- Run 2: differently shaped brief ("Your job" / "Definition of done" / "Report back") that
  **omitted `IMPLEMENTATION.md`** and told the implementer to "return findings as your final
  message … do not write report files" — directly contradicting the implementer persona,
  which writes `IMPLEMENTATION.md`. A pipeline-contract divergence, not just cosmetic.

### WITH arm (convergence)
Produced the fixed sections from the recipe: reference paths only; failing test + the exact
`cd backend && pytest bookings/tests/test_pricing.py -x` command lifted verbatim; File plan
table transcribed; `Required fixes: None — verdict PASS.`; N1 under `Deferred — do NOT act on`;
iteration cap + escalation files; and the `On completion` block (write `IMPLEMENTATION.md`,
leave uncommitted, orchestrator owns the commit) — closing the run-2 contradiction.

## Verdict
The recipe's win is determinism and a stable pipeline contract, not "stop pasting bodies"
(agents already largely don't). It eliminates the obligation-set drift — notably the run-2
brief that would have broken the implementer's `IMPLEMENTATION.md` hand-off. Ship.
