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
- **The WITHOUT arm is not a blank slate — say what it already knows.** Subagents inherit the
  session's global `CLAUDE.md`, and through it `~/.claude/shared/working-agreements.md`. So an
  arm run without the skill may still hold a weaker written form of the same rule, and in the
  2026-10-01 comment-pass runs both WITHOUT arms cited the working agreements by name. That is
  often the *right* comparison — "the rule as already written" vs. "the rule as a procedure" is
  exactly the question when a standard exists and is being ignored — but it is a different claim
  from "rule vs. no rule", and a result that doesn't name which one it measured will be read as
  the stronger one. Name it in the result file.
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
- **A fixture that the baseline passes has told you something — write it down and keep it.**
  The outcome table says "sharpen the scenario, or question whether the skill earns its
  context", and both are real options, but a third is common: the fixture is measuring a
  different stage than the one that fails. `comment-altitude-pass` measures whether the
  *backstop* can tell a keep from a delete; the reported failure was at write time. Keep such a
  fixture as a regression test for the stage it does cover, and say in its result file that it
  is not evidence for the change that prompted it.
- **When the pass criteria don't move, check whether anything else did.** The comment-pass WITH
  arms produced counts and a per-comment verdict where the WITHOUT arms produced prose: equally
  good code, and only one version a reviewer can check without redoing the work. Auditability,
  determinism and hand-off quality are real deltas that no code-shaped criterion will catch.
  Report them as what they are rather than promoting them into a pass.
