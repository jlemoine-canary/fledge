# evals

Behavioral fixtures for fledge's own skills. Each eval is a controlled A/B: run a
scenario through a subagent *without* the skill and *with* it, and compare against the
fixture's observable pass criteria. The delta is the evidence the skill changes behavior.

This is the lightweight harness referenced by `references/skill-eval-protocol.md`. It is
**Claude-Code-native** — runs entirely through the `Agent` tool, no scripts, no runtime
dependency. Fledge stays a pure plugin.

## Layout

```
evals/
├── README.md                       # this file
├── _template/
│   ├── scenario.md                 # copy this to author a new fixture
│   └── result.md                   # copy this to record a run
└── <skill>/
    └── <scenario>/
        ├── scenario.md             # the fixture (task, setup, pass criteria, failure modes)
        └── result-<run-label>.md   # one per run (e.g. result-2026-06-25.md)
```

## Running an eval

Use the skill:

```
/fledge:fledge-eval <skill> [<scenario>] [--mode delta|red|green]
```

It loads the fixture, spawns the two arms, and writes a result file. See
`references/skill-eval-protocol.md` for the full mechanics and how to read the outcome.

## Authoring a fixture

Use `/fledge:fledge-writing-skills` (RED stage) and copy `_template/scenario.md`. A
fixture must be self-contained and re-runnable, with **observable** pass criteria — you
must be able to point at a transcript line or produced artifact and call pass/fail.

## Seed fixtures

| Skill | Scenario | What it checks |
|---|---|---|
| `fledge-implement` | `comment-rot-at-write-time` | The implementer doesn't *author* rot-prone comments — no docstring pinning the return shape, no enum values copied into prose — while still commenting the one constant whose value is unguessable. |
| `fledge-implement` | `comment-altitude-pass` | Review-side backstop: given a file full of bad comments, the cleanup pass tells a keep from a delete instead of purging. Baseline passes this one — see its result file. |
| `fledge-ingest` | `prior-design-search` | Ingest searches Slack, Notion and Linear for earlier investigations of the problem and records them as `kind: prior-work`, not just the sources the user supplied. |
| `fledge-plan` | `state-changes-and-naming` | A plan for an owner/suppression gate lists each state change with its outcome and settles new names before the migration unit. |
| `fledge-plan` | `stale-sot` | The planner raises a SoT that predates and disagrees with the project description as an open question. Baseline passes; kept as a regression test. |
| `fledge-review` | `stale-sot-and-state-gaps` | Plan review treats a SoT older than, and disagreeing with, the project description as an `Open questions` item (P1), and asks for state changes and names (P12). |
| `fledge-test` | `private-function-test` | The agent tests behavior through the public API instead of calling a private `_helper()` directly. |
| `fledge-writing-skills` | `no-baseline-failure` | The agent refuses to write a skill until it has documented a real baseline failure (no skill from imagination). |
