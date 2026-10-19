# Scenario: fledge-lite / small-ticket-overhead

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-lite` — whether a one-ticket, one-surface, ~80-line fix gets a pipeline sized to it
instead of the full pipeline's ingest manifest, planner subagent, three-round plan review and
two checkpoints.

Both arms get the fledge plugin. The WITHOUT arm gets it **without** `skills/fledge-lite/`; the
WITH arm gets the same tree **with** it. This measures "fledge as it is" vs "fledge plus a lite
path", not "no skill" vs "skill" — the baseline is a working pipeline that is simply the wrong
size for this ticket.

## Task prompt

> You are running the fledge pipeline on a single Linear ticket, **MSG-6639**. It is the only
> source of truth; the user supplied nothing else.
>
> Read `<SKILL_ROOT>/skills/` and the references it points to (paths relative to `<SKILL_ROOT>`),
> and decide how this ticket should be run.
>
> **This is a sandbox. Do not call any MCP tool, reach any network service, or write any code.**
> The connected services are simulated by files under
> `evals/fledge-lite/small-ticket-overhead/workspace/`:
> - Fetching the Linear ticket = `Read` `workspace/linear/MSG-6639.md`.
> - Fetching the linked Sentry event = `Read` `workspace/sentry/event-4471.md`.
> - There is no repository to inspect and no tests to run. Where a stage would touch code,
>   say what it would do rather than doing it.
>
> Produce an **execution plan** and nothing else. It must state, explicitly and countably:
> 1. Every stage you will run, in order.
> 2. Every subagent you will spawn, by persona name — one line each. Give the total count.
> 3. Every point at which you will stop and ask the user. Give the total count.
> 4. How many review rounds the code review gets.
> 5. Any condition that would make you stop and run something different instead.
>
> Do not ask the user anything; every answer you need is above.

## Setup / context

- `workspace/linear/MSG-6639.md` — the ticket. Single surface (`frontend/` only), no API
  change, one source of truth, a ~4-bullet fix.
- `workspace/sentry/event-4471.md` — the linked stack trace naming the exact failing site.
- `<SKILL_ROOT>` — a copy of the fledge plugin tree. WITHOUT arm: `skills/fledge-lite/` removed.

## Pass criteria (observable)

Point at the produced execution plan for each.

1. **Spawn count ≤ 2.** The stated total number of subagents is 2 or fewer.
2. **No planner subagent.** `fledge-planner` is not in the spawn list; the plan is written by
   the orchestrator itself.
3. **No implementer subagent.** `fledge-implementer` is not in the spawn list.
4. **One review round, adversarial only.** The code review is stated as a single round with
   `fledge-reviewer-adversarial`; `fledge-reviewer-constructive` and `fledge-reviewer-integrator`
   do not appear.
5. **Exactly one user checkpoint**, and it is the final "Ship it?" — no plan-approval checkpoint.
6. **No source manifest stage.** The plan does not produce `.fledge/SOURCES.md` or run
   `/fledge:fledge-ingest`.
7. **Pre-flight survives.** The plan still names the dedicated worktree off the default branch,
   the `<git-user>/<TICKET>/<slug>` branch, the stale-state check, and GPG verification.
8. **Mutation check is present and mandatory.** The plan names a step that breaks the fix on
   purpose (2–3 targeted mutations) and confirms a test goes red for each — not "mentally
   mutation-test", not optional.
9. **Escalation trip-wires are named before planning.** The plan states the conditions that
   abort the small path and hand back to full `/fledge:fledge`, and places that check before the
   plan is written.
10. **Closing report.** The plan ends with a report naming what was kept and what was skipped
    relative to the full pipeline.

## Known failure modes (what the WITHOUT arm is expected to do)

- Runs `/fledge:fledge` end to end: ingest → plan → review(plan) → test → implement → review(code)
  → qa, ~10 subagent spawns.
- Spawns `fledge-planner` for a four-bullet plan.
- Runs the three-round review (constructive → adversarial → integrator), on the plan *and* again
  on the code.
- Two checkpoints: plan approval and ship.
- Writes `.fledge/SOURCES.md` for a single source.
- Treats mutation testing as a line inside the test stage ("mentally mutation-test") rather than
  a mandatory step with its own evidence.
- States no condition under which it would switch pipelines — there is nothing to switch to.

## Notes

- Ticket and Sentry content are **synthetic**, written for this fixture. The baseline they model
  is a real run (an ~80-line single-surface frontend fix) whose cost prompted the skill; the
  fixture reproduces the shape, not that ticket's content.
- Criteria 1–6 measure the trim. Criteria 7–10 are the guard: a lite path that drops the
  pre-flight, the mutation check or the escalation wires is worse than the full pipeline, not
  cheaper. Score them separately — a run that passes 1–6 and fails 7–10 is a **fail**.
- Subagent output is stochastic. For a borderline spawn count, run each arm twice and report the
  spread.
