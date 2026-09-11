# Scenario: fledge-plan / pr-slice-declaration

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-plan` (and `references/templates/plan.md`) — whether a plan for a phase that is
obviously several shippable pieces **declares those pieces as separate pull requests**, or
plans the phase as one undifferentiated deliverable.

## Task prompt

> You are planning one phase of work against a large Django + Vue monorepo. Produce the
> `## File plan` and any scoping sections you think a staff engineer would include, using the
> fledge plan template at `references/templates/plan.md`.
>
> **Ticket CC-3083 — "B2B: chain selector, simple-path detection and bounds"**
>
> Reservations at a hotel can form a "back-to-back chain": the same guest checking out and
> immediately checking into another room, one or more times. We need to detect these runs.
>
> Deliver all of:
> 1. Move the reservation-grouping vocabulary (a `GUEST_JOURNEY_ELIGIBLE_STATUSES` set and a
>    `display_name` helper) out of `guest/services/reservation.py` into the model layer at
>    `guest/models/reservation.py`. Behavior-neutral.
> 2. Add `ReservationSelector.get_back_to_back_chain(reservation) -> BackToBackChain` in
>    `guest/selectors/reservation.py`, ordered by arrival, exposing `first_leg` / `last_leg` /
>    `span`. This is the bulk of the work: candidate loading, chain resolution, bounds, and
>    exclusion logging.
> 3. Memoize chain resolution per evaluation scope so repeated lookups in one scheduling pass
>    don't re-query.
> 4. Add thin `Reservation.is_back_to_back` and `Reservation.back_to_back_chain` properties
>    that delegate to the selector.
>
> There are no consumers of any of this yet — a later ticket wires it into scheduling.
>
> The ticket text above **is** the source-of-truth for this task. Do not fetch it from Linear or any
> other external system, and do not look for a live version — plan against exactly what is written here.

## Setup / context

- The fledge plan template: `references/templates/plan.md`.
- Treat the repo as a mature monorepo with a conventional Django app layout, a test suite per
  app, and code review by humans.
- No other fledge artifacts are needed; the arm should produce plan sections, not code.

## Pass criteria (observable)

1. The plan contains a section that **names two or more separate pull requests** (or
   equivalently-named shippable units) and assigns each file/deliverable to exactly one of them.
2. Each declared PR has a stated **landing order or dependency** relative to the others.
3. Each declared PR is justified by something concrete — reviewability, an independent revert
   boundary, a lint/migration rule, or a deploy-ordering constraint — not "it felt big".
4. The memoization piece (deliverable 3) is either its own PR or explicitly deferred, rather
   than folded silently into the selector PR — it has no consumer, so it is separable.
5. The `File plan` rows are attributable to a declared PR (a column, a grouping, or an explicit
   statement that all rows land together and why that is acceptable).

## Known failure modes (what the WITHOUT arm is expected to do)

- Produces one `## File plan` table covering all four deliverables with no PR boundaries at all,
  implying a single branch and a single review.
- Uses `## Sub-phases` to split *planning* work but still implies one landing.
- Mentions "small PRs" as generic advice without assigning specific files to specific PRs.
- Folds the memoization into the selector because they touch the same file.

## Notes

- This scenario reproduces a real outcome. CC-3083 was hand-sliced by the user into four PRs;
  the three small ones (+69, +133, +87) merged within 0–1 days and the un-sliced selector PR
  (#53519, +1610/−9 across 2 files, 337 production lines and 1273 test lines) sat open 24 days
  and drew the reviewer comment "I would consider starting without memoization, especially since
  there are no consumers yet."
- **Hermeticity matters.** The 2026-09-11 run was not perfectly controlled: the WITHOUT arm had no
  Linear access and the WITH arm did, so one arm saw live ticket context the other never did. MCP
  availability varies by session, so the prompt now forbids external fetching. If an arm reports
  fetching the live ticket anyway, discard that run.
- Stochastic. Run 2–3 times per arm before trusting a borderline result; the failure here is an
  omission, so a single run that happens to mention PRs is not a pass unless criteria 1–5 hold.
