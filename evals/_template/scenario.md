# Scenario: <skill> / <scenario-name>

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable. Copy this file to
> `evals/<skill>/<scenario>/scenario.md` and fill every section.

## Skill under test
`<skill-name>` — the behavior this scenario exercises: <one line>.

## Task prompt
<The exact instruction handed to BOTH arms, verbatim. Self-contained. Must NOT mention
"the skill" or its rules — the WITHOUT arm has to be able to fail naturally.>

## Setup / context
<Files, code snippets, or fixtures the subagent needs to do the task. Inline them here or
point to paths within the repo. Both arms get exactly this.>

## Pass criteria (observable)
<Numbered, each one a check you can point at a transcript line or produced artifact for.>
1. <e.g. The produced test asserts via a public method, not a name-mangled/underscore-prefixed member.>
2. ...

## Known failure modes (what the WITHOUT arm is expected to do)
<The specific wrong behaviors the skill exists to prevent.>
- <e.g. Calls `obj._compute()` directly to assert the private result.>

## Notes
<Stochasticity warnings, how many runs to average, anything that makes a run honest.>
