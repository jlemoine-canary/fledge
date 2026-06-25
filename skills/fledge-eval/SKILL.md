---
name: fledge-eval
description: Use when the user says "eval <skill>", "does this skill work", "test the skill", after authoring/revising a skill via /fledge:fledge-writing-skills, or to capture a RED baseline before a change. Runs a fixture scenario through a without-skill subagent and a with-skill subagent and reports the behavioral delta against the fixture's pass criteria — Claude-Code-native, no scripts.
---

# fledge-eval

The lightweight harness behind fledge's skill-authoring loop. It answers one question: **does this skill change what the agent does?** It does that the Claude-Code-native way — by spawning two subagents on the same scenario, one without the skill and one with it, and comparing their behavior against the fixture's pass criteria. No external scripts, no new runtime dependency.

Read `references/skill-eval-protocol.md` — that is the source of truth for the protocol; this skill executes it.

## Preconditions

- A fixture exists at `evals/<skill>/<scenario>/scenario.md` (use `/fledge:fledge-writing-skills` and `evals/_template/` to author one). If it doesn't, this skill helps you write it first — a scenario with no fixture cannot be re-run or compared.

## Arguments

- **Skill name** — e.g. `fledge-test`. Required.
- **Scenario** — fixture subdirectory under `evals/<skill>/`. If omitted, list available scenarios and ask; if there's exactly one, use it.
- **`--mode`** — `delta` (default: run both arms, compare), `red` (without-skill only — capture/refresh the baseline), or `green` (with-skill only — confirm a fix).

## Process

### 1. Load the fixture
Read `evals/<skill>/<scenario>/scenario.md`. It defines: the task prompt, the setup context, the **observable pass criteria**, and the **known failure modes** the skill is meant to prevent. If pass criteria aren't observable (you can't point at a transcript line and say pass/fail), the fixture is defective — fix it before running.

### 2. Run the WITHOUT-skill arm (RED)
Spawn a fresh subagent with the scenario's task prompt and setup, but **do not** give it the skill under test or mention the skill's rules. This is the baseline — how the agent behaves naturally.
- Capture the relevant transcript excerpt (the artifact it produced, the decision it made).
- Score it against the pass criteria. The baseline is *expected to fail* — if it passes, the skill may be redundant; note that.

### 3. Run the WITH-skill arm (GREEN)
Spawn a second fresh subagent on the **same** task prompt and setup, this time instructing it to follow the skill (pass the skill's path/content, or invoke it). Same scoring.

Keep the arms honest: identical task + setup, fresh context each, the *only* deliberate difference is the skill's presence. Vary nothing else.

### 4. Report the delta
Write `evals/<skill>/<scenario>/result-<run-label>.md` from `evals/_template/result.md`. Record:
- Per-arm pass/fail against each criterion.
- The **delta**: which failures the skill flipped to passes (the evidence it works), and any failure the skill did *not* fix (the next thing to close in REFACTOR).
- Any new rationalization the with-skill arm invented to route around the rule — these feed back into `/fledge:fledge-writing-skills` REFACTOR.

Output a short summary to the user: `delta: N/M criteria fixed by the skill` plus the unfixed ones.

## Error handling
- **Subagent can't run the scenario** (missing setup, ambiguous prompt) → the fixture is underspecified; fix the fixture, don't fudge the result.
- **Both arms pass** → either the skill is redundant or the scenario doesn't exercise the failure mode. Sharpen the scenario or question the skill.
- **Both arms fail** → the skill didn't change behavior. Back to `/fledge:fledge-writing-skills` GREEN — the instruction is too weak or buried.

## What this skill does NOT do
- Author or edit the skill under test — that's `/fledge:fledge-writing-skills`. This skill only measures.
- Modify production/project code — it runs scenarios in throwaway subagent context.
- Guarantee a verdict from a single run — subagent behavior is stochastic; for a close call, run the arms more than once and say so.

## Tools needed
- `Read`, `Write` (fixtures and results)
- `Agent` (the two arms)
- `AskUserQuestion` (scenario selection when ambiguous)

## Related
- Authoring loop: `/fledge:fledge-writing-skills`
- Protocol source of truth: `references/skill-eval-protocol.md`
- Fixtures: `evals/`
