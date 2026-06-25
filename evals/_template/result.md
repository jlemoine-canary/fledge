# Eval result: <skill> / <scenario> — <run-label>

> Produced by `/fledge:fledge-eval`. One file per run. Copy to
> `evals/<skill>/<scenario>/result-<run-label>.md`.

- **Scenario:** `evals/<skill>/<scenario>/scenario.md`
- **Mode:** delta | red | green
- **Runs per arm:** <N> (subagent output is stochastic; >1 for borderline calls)

## Scorecard
| Pass criterion | WITHOUT arm | WITH arm |
|---|---|---|
| 1. <criterion> | fail | pass |
| 2. ... | | |

## Delta
- **Fixed by the skill:** <criteria the WITH arm passed that the WITHOUT arm failed — the evidence>
- **Not fixed:** <criteria the WITH arm still failed — feeds REFACTOR>
- **New rationalizations observed:** <evasions the WITH arm invented; feed back to fledge-writing-skills>

## Transcript excerpts
### WITHOUT arm
<the artifact / decision that earned the score>

### WITH arm
<same>

## Verdict
<one line: does the skill change behavior in the intended direction? what's the next step?>
