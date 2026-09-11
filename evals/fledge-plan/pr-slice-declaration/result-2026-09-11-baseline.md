# Result: fledge-plan / pr-slice-declaration — WITHOUT arm (RED)

**Date:** 2026-09-11 · **Plugin version under test:** 0.5.1 · **Arm:** WITHOUT (baseline)
**Runs:** 1 controlled subagent, corroborated by 4 production `PLAN.md` artifacts (see Corroboration).

## Verdict: FAIL — 2 of 5 criteria

| # | Criterion | Result |
|---|---|---|
| 1 | Names ≥2 separate PRs, each deliverable assigned to exactly one | **FAIL** |
| 2 | Each declared unit has a landing order / dependency | PASS |
| 3 | Each split justified by something concrete | **PARTIAL** |
| 4 | Memoization its own unit or explicitly deferred | PASS |
| 5 | `File plan` rows attributable to a declared PR | **FAIL** |

## What the baseline actually did

It produced a `## Sub-phases` section — `CC-3083.1` vocabulary move, `.2` chain selector, `.3` model
properties, `.4` memoization — each with a `Depends on:` line. Criterion 2 passes cleanly. Criterion 4
passes too: Q6 recommends deferring memoization outright, citing "C12 names memoization as the most
common instance of machinery without a caller." **That is 0.5.0's new checklist item working**, and it
is the single finding the human reviewer raised on the real PR (#53519).

The failure is narrower and more interesting than "the planner doesn't think about splitting":

- **It split for *planning*, not for *landing*.** Only `.1` mentions a PR at all ("Ships as its own
  commit/PR — a pure-move diff is reviewable at a glance"). `.2` — the sub-phase corresponding to the
  real +1610-line PR that sat open 24 days — gets **no PR-boundary reasoning whatsoever**. Sub-phases
  are units of planning work; nothing in the plan says how many times this phase reaches `main`.
- **The `File plan` is one undifferentiated table.** Every row spans all four deliverables with no
  attribution, and the `guest/models/reservation.py` row carries two different sub-phases in one cell
  ("Receive `GUEST_JOURNEY_ELIGIBLE_STATUSES` … Later sub-phase: add `is_back_to_back` …"). The File
  plan is the artifact the implementer consumes, so even a good sub-phase split does not reach the
  branch.
- **Nothing downstream could act on it anyway.** `fledge-implement` §0 creates exactly one branch per
  phase — `<git-user>/<TICKET-ID>/<phase-slug>` — so all four sub-phases land on one branch and become
  one PR by construction.

## Corroboration from production

Grep for PR-slicing language across all of fledge 0.5.1 (`skills/`, `agents/`, `references/`):
**zero matches** for slice / shippable / "one PR per" / "separate PRs".

The four real `PLAN.md` artifacts on disk:

| Run | `## Landing`-equivalent section? | PR mentions |
|---|---|---|
| `01-email-sidebar-nav` (CC-2804) | none | 1 |
| `01-email-islands-layout` (CC-3022) | none | 0 |
| `01-twilio-message-admin-commands` (CC-2794) | none | 5 |
| `01-span-date-proxy` (CC-3084) | none | 18 |

The last row is the informative one. That plan *did* work out a two-PR split — but only because the
project's own lint rule (`scripts/linter/rules/migration_files_only.ts`, a 20-added-line budget for
non-allowlisted files) made it mandatory, and it wrote the result into `PRECONDITION-PRS.md`, an
**ad-hoc file no fledge skill defines, no template names, and no reviewer knows to look for.** So the
planner will reason about PR boundaries when an external constraint forces it, and has nowhere to put
the answer. Reviewability alone never forces it.

## Known failure modes — observed

- ✅ "Uses `## Sub-phases` to split *planning* work but still implies one landing." — exactly this.
- ✅ "Produces one `## File plan` table … with no PR boundaries."
- ❌ "Mentions 'small PRs' as generic advice" — did better than this; `.1` and `.3` have real
  justifications. The gap is that the *largest* unit has none.
- ❌ "Folds the memoization into the selector" — did not; C12 caught it.

## Conclusion for GREEN

The fix is not "tell the planner to make small PRs" — it already believes that. The fix is:

1. Give the plan a **named home for the landing split**, distinct from `## Sub-phases`, so the answer
   stops landing in ad-hoc files.
2. Make the **`File plan` rows attributable** to a landing unit, since that is the table the
   implementer reads.
3. Make **`fledge-implement` branch per landing unit** instead of per phase — otherwise the landing
   plan is machinery without a caller, which C12 would reject.
