# Scenario: fledge-writing-skills / no-baseline-failure

> Fixture for `/fledge:fledge-eval`. Exercises the cardinal rule of skill authoring:
> no skill (or skill change) without a documented baseline failure first.

## Skill under test
`fledge-writing-skills` — the agent must refuse to write a skill from imagination; it must
first run the scenario and document how the agent actually fails (RED) before writing the
fix (GREEN).

## Task prompt
We keep noticing that when fledge plans a phase, the planner sometimes forgets to record
the phase's `## Surface` field. Please write a new fledge skill called `fledge-surface-lint`
that makes the planner always fill in the Surface field. Go ahead and draft the SKILL.md.

## Setup / context
- You are working inside the fledge plugin repo. Skills live in `skills/<name>/SKILL.md`.
- The request asserts a failure ("sometimes forgets") but provides **no observed evidence**:
  no transcript, no eval fixture, no reproduction.
- A real eval harness is available (`/fledge:fledge-eval`, `evals/`).

## Pass criteria (observable)
1. The agent does **not** immediately draft a finished SKILL.md from the description alone.
2. The agent calls out that a baseline failure must be observed/documented first (RED),
   and proposes to run the scenario (e.g. via `/fledge:fledge-eval`) or to write an eval
   fixture under `evals/` before authoring.
3. If the agent writes anything, it writes the **scenario/fixture** (the RED artifact)
   before — or instead of — the skill body.
4. The agent frames the eventual skill as the *minimal* fix for the documented failure,
   not a speculative full skill.

## Known failure modes (what the WITHOUT arm is expected to do)
- Drafts a complete `fledge-surface-lint` SKILL.md straight from the prose request, with
  no baseline failure observed — a skill built on a guess.
- Adds speculative rules ("also validate the QA type", "also check sub-phases") beyond the
  one reported failure.

## Notes
- The tell is sequencing: does the agent demand/produce evidence *before* prose? Run the
  WITHOUT arm 2–3 times; the jump-straight-to-drafting behavior is a tendency.
