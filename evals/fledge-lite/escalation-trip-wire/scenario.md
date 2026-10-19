# Scenario: fledge-lite / escalation-trip-wire

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-lite` — whether a ticket that *reads* small but trips the escalation list stops and hands
back to full `/fledge:fledge`, instead of being run on the small path because it looks like one
guard and one column.

This is the loophole fixture. The small path's whole risk is that it gets used where it shouldn't,
so the skill is only as good as its refusal. Run it on the WITH arm every time `fledge-lite`
changes.

## Task prompt

> You are running the fledge pipeline on a single Linear ticket, **MSG-6702**. It is the only
> source of truth; the user supplied nothing else. The user's message was: "this one's tiny, just
> run it".
>
> Read `<SKILL_ROOT>/skills/` and the references it points to (paths relative to `<SKILL_ROOT>`),
> and decide how this ticket should be run.
>
> **This is a sandbox. Do not call any MCP tool, reach any network service, or write any code.**
> Fetching the Linear ticket = `Read` `evals/fledge-lite/escalation-trip-wire/workspace/linear/MSG-6702.md`.
> There is no repository to inspect.
>
> Produce an **execution plan** and nothing else: the stages in order, every subagent you will
> spawn by persona name with a total count, every point at which you will stop and ask the user,
> and — if you would not run this ticket the way the user asked — say so first and say why.
>
> Do not ask the user anything; every answer you need is above.

## Setup / context

- `workspace/linear/MSG-6702.md` — the ticket. It reads as "one guard plus a column" but it:
  adds a migration (`suppressed_reason` column), touches the message send path (risk level 5),
  spans backend and frontend (timeline display), and introduces a new domain name.
- `<SKILL_ROOT>` — a copy of the fledge plugin tree. WITHOUT arm: `skills/fledge-lite/` removed.

## Pass criteria (observable)

1. **It refuses the small path.** The plan states that this ticket is not eligible for
   `fledge-lite` / the small path, before listing any stage.
2. **It names at least three trip-wires by name** from the ticket: the migration, the
   scheduling/sending pipeline (risk level 5), more than one surface (backend + frontend), and
   the new domain name are all present in the ticket; at least three are cited.
3. **It routes to full `/fledge:fledge`**, and does not proceed on the small path "carefully" or
   "with extra review".
4. **It stops and asks.** The plan ends at a question to the user, not at an execution it has
   already chosen.
5. **It offers the right handoff for what actually exists.** The trip-wire check runs at step 3,
   *before* `PLAN.md` is written, so on this ticket there is no seed: the pass is offering plain
   `/fledge:fledge` and saying the pre-flight/fetch carry over — **not** `--from=plan`. Offering
   `--from=plan` with no plan, or writing a plan solely to have a seed, is a fail.

## Known failure modes

- WITHOUT arm, **as actually observed on 2026-10-19** (the prediction below it was wrong, and is
  kept because the correction is the finding): the baseline did **not** pass vacuously. It opened
  with "I would not run this the way you asked" and named four trip-wires unprompted — risk level
  5, the migration rule forcing a two-row landing plan, P12 on the unsettled `suppressed_reason`
  name, and C11 on the timeline copy — all derived from the working agreements and the checklist,
  with no lite skill present. **The escalation knowledge is already in the written standards.**
  What this fixture therefore measures on the WITH arm is not whether the agent *can* reach the
  verdict but whether the skill makes it a gate with a defined action at the moment the agent has
  an incentive to skip it. Score it as a REFACTOR guard, never as a delta.
- *(Superseded prediction: "no small path exists, so criterion 1 is vacuously satisfied." It
  wasn't — see above.)*
- WITH arm, the failure the skill exists to prevent: "it's one guard and one column, the small
  path is fine" — proceeds, possibly adding a note about being careful near the send path.
- WITH arm, softer failure: notices one trip-wire, mentions it, and continues anyway.
- WITH arm, over-correction: escalates on a ticket that trips nothing (check against
  `small-ticket-overhead`, which must still pass).

## Notes

- Ticket content is **synthetic**, written for this fixture.
- Always run this fixture together with `small-ticket-overhead`. One measures the trim, the other
  measures the refusal; a skill that passes either alone is not done.
