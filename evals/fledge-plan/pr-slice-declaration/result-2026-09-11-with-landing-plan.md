# Result: fledge-plan / pr-slice-declaration — WITH arm (GREEN)

**Date:** 2026-09-11 · **Change under test:** `## Landing plan` section + `File plan` PR column
(`references/templates/plan.md`), planner instruction, `fledge-plan` skill framing.
**Runs:** 1. Compare against `result-2026-09-11-baseline.md`.

## Verdict: PASS — 5 of 5 (was 2 of 5)

| # | Criterion | RED | GREEN |
|---|---|---|---|
| 1 | Names ≥2 separate PRs, each deliverable assigned to one | FAIL | **PASS** |
| 2 | Landing order / dependency per unit | PASS | PASS |
| 3 | Each split concretely justified | PARTIAL | **PASS** |
| 4 | Memoization its own unit or explicitly deferred | PASS | PASS |
| 5 | `File plan` rows attributable to a declared PR | FAIL | **PASS** |

## The delta

**Criterion 1 + 3.** The arm produced a `## Landing plan` table of four units with a reason per row.
PR1 — "Pure-move diff, reviewable at a glance, and unambiguous if something breaks. It also touches
files outside the feature (every importer), which is exactly the content you want out of the
substantive diff. Independent revert boundary." PR2 — "The substantive change — reviewed on its own
rather than buried behind a move." Compare the baseline, where the equivalent unit (`.2`, the real
+1610-line PR) carried **no PR-boundary reasoning at all**.

**Criterion 5.** Every `File plan` row now opens with its PR. The baseline's collapsed cell
("Receive `GUEST_JOURNEY_ELIGIBLE_STATUSES` … Later sub-phase: add `is_back_to_back` …") became two
rows under PR1 and PR4 — the template's "a row that belongs to two units is two rows" rule firing.

**The rule did not over-fire.** The arm wrote: "PR2 is the large one and that is fine — size is a
symptom, not the criterion, and it is one coherent change." That is the guard against the obvious
failure mode of this change, working. A rule that produced four equal-sized PRs here would have been
worse than the baseline.

**Attribution is unambiguous** for criteria 1/3/5: the arm used the template's exact section name,
its exact column headers (`PR | Contents | Depends on | Why its own PR`), and quoted its rule text
back. None of that exists in 0.5.1.

## Validity caveat — the arms were not perfectly controlled

The WITHOUT arm reported "the Linear MCP servers failed to connect this session." The WITH arm's
Linear access worked, and it pulled the live ticket (`CC-3083` → `MSG-6118`), surfacing real context
the baseline never saw: the model accessors are listed out of scope with a follow-up ticket, and the
memo is a recorded no-op across the bulk path's worker pool. **That is an uncontrolled difference
between arms**, and it plausibly made the WITH arm more inclined to separate deliverables 3 and 4.

It does not explain criteria 1, 3 and 5, which are structural — the template's section name, its
column headers, and its verbatim rule text are not derivable from a Linear ticket. Criterion 4 passed
in **both** arms anyway (C12 caught it in the baseline), so the Linear context changed nothing there.

**Fixture defect to fix before the next run:** the scenario is not hermetic. An arm that can reach
Linear gets different material than one that cannot, and MCP availability varies by session. The
prompt should forbid external source fetching and state that the ticket text in the fixture *is* the
source-of-truth. Filed as a note in `scenario.md`.

## Incidental finding (not this fixture's business)

The live ticket says the model-level accessors were "attempted and dropped — CC-3271" and the
`ContextVar` memo "does nothing across the bulk path's 20-worker thread pool." Both map to open PRs
in the real CC-3083 stack (#53521 closed, #53519 open 24 days). Worth a look independently of fledge.
