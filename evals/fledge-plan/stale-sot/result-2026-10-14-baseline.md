# Eval result: fledge-plan / stale-sot — 2026-10-14 baseline

- **Scenario:** `evals/fledge-plan/stale-sot/scenario.md`
- **Mode:** red (WITHOUT arm, the 0.7.0 planner). A WITH arm was run against a draft planner
  sentence and passed too. That sentence was then dropped, because this baseline passed, so the WITH
  run measures text that didn't ship and isn't scored.
- **Runs:** 1. Working agreements present; the arm read them.

## Scorecard
| Pass criterion | WITHOUT arm |
|---|---|
| 1. Read S2, not just the SoT | pass |
| 2. `Open questions` names the S1/S2 disagreement | pass ("Q1: which source wins") |
| 3. Doesn't silently pick a side | pass (planned to S1 as the manifest requires, and marked the plan BLOCKED on Q1) |

## Verdict
The baseline passes. The 0.7.0 planner already reads the manifest's other sources and raises the
conflict. So the retro's "fledge trusts the SoT" failure did not reproduce at plan time with a
manifest that lists both sources. No planner change was made for it. Keep this fixture as a
regression test.

One caveat: the prompt explains how to fetch each manifest source, S2 included, which may have
nudged the arm to read S2. A run whose prompt names only the manifest path would test that more
strictly.
