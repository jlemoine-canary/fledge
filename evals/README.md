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
| `fledge-test` | `private-function-test` | The agent tests behavior through the public API instead of calling a private `_helper()` directly. |
| `fledge-writing-skills` | `no-baseline-failure` | The agent refuses to write a skill until it has documented a real baseline failure (no skill from imagination). |
