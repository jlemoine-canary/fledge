# Skill eval protocol

How fledge measures whether a skill actually changes subagent behavior. This is the
single source of truth for the eval mechanics; `skills/fledge-eval/SKILL.md` executes
it and `skills/fledge-writing-skills/SKILL.md` cites it for the RED/GREEN/REFACTOR loop.

The protocol is **Claude-Code-native by design**: it runs entirely through the `Agent`
tool. There are no shell or Python scripts and no new runtime dependency — fledge stays
a pure plugin. The "harness" is a disciplined way of spawning subagents and comparing
them, not a program.

## The core idea: a controlled A/B on behavior

A skill earns its place only if it changes what the agent does. So we run the same
scenario twice, changing exactly one thing — the skill's presence:

- **WITHOUT arm (RED / baseline):** a fresh subagent gets the task and setup, but not
  the skill or its rules. This is how the agent behaves naturally.
- **WITH arm (GREEN):** a second fresh subagent gets the *same* task and setup, plus the
  skill. 

The **delta** between the arms — which failure modes the skill flipped to passes — is the
evidence the skill works. No delta means no skill.

### Keep the arms honest
- **Identical task + setup.** Copy the prompt verbatim into both arms. The only deliberate
  difference is the skill.
- **Fresh context per arm.** Each arm is a new subagent; no shared memory, no leakage of
  the rule from one arm into the other.
- **Don't coach the WITHOUT arm.** It must be allowed to fail naturally — that failure is
  the whole point of RED.
- **Stochasticity is real.** Subagent output varies run to run. For a borderline result,
  run each arm 2–3 times and report the spread rather than a single sample.

## Fixture anatomy

A fixture lives at `evals/<skill>/<scenario>/scenario.md` and must be self-contained and
re-runnable. See `evals/_template/scenario.md`. Required parts:

| Part | What it is |
|---|---|
| **Task prompt** | The exact instruction handed to both arms. Self-contained — no reference to "the skill". |
| **Setup / context** | Files, snippets, or fixtures the arm needs to do the task. Inlined or pointed to within the repo. |
| **Pass criteria** | **Observable** checks — you can point at a transcript line or produced artifact and call pass/fail. ("The test exercises behavior through a public method", not "the test is good".) |
| **Known failure modes** | The specific wrong behaviors the skill is meant to prevent — what you expect the WITHOUT arm to do. |

If a pass criterion isn't observable, the fixture is defective. Fix the fixture before
trusting any result from it.

## Running an eval

1. **Load the fixture.** Validate the pass criteria are observable.
2. **RED — WITHOUT arm.** Spawn a fresh subagent with task + setup only. Capture the
   artifact/decision. Score against the criteria; expect failure.
3. **GREEN — WITH arm.** Spawn a fresh subagent, same task + setup, plus the skill. Score.
4. **Report the delta.** Write `result-<run-label>.md` from `evals/_template/result.md`:
   per-arm scores, the criteria the skill fixed, the criteria it *didn't*, and any new
   rationalization the WITH arm invented to dodge the rule.

## Reading the result

| Outcome | Meaning | Next step |
|---|---|---|
| WITHOUT fails, WITH passes | The skill works — this is the win. | Record the delta as evidence; ship. |
| WITHOUT passes, WITH passes | Skill may be redundant (agent already does it). | Sharpen the scenario, or question whether the skill earns its context. |
| WITHOUT fails, WITH fails | Skill didn't change behavior. | REFACTOR: strengthen/relocate the instruction (`fledge-writing-skills` GREEN). |
| WITH invents a new dodge | Skill leaks. | Add a "Rationalizations to reject" counter; re-run (`fledge-writing-skills` REFACTOR). |

## Scope and honesty

- This is a **lightweight** harness on purpose. It catches whether a skill moves behavior
  in the intended direction on a few seed scenarios — not a statistical guarantee.
- A fixture is a regression test for *process*. When a skill change fixes a real failure,
  add a fixture so the failure can't silently return.
- Never report a delta you didn't observe. A run that couldn't complete is an
  underspecified fixture, not a pass or a fail.
