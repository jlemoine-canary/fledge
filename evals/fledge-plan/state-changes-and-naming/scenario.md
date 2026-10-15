# Scenario: fledge-plan / state-changes-and-naming

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-planner` + `references/templates/plan.md` — whether a plan for an owner/suppression gate
**lists each state change and its outcome**, and **settles new domain names** before the migration
landing unit, rather than leaving both to code review.

Both arms run the planner persona and plan template; the WITHOUT arm gets the 0.7.0 versions
(`<SKILL_ROOT>` = snapshot), the WITH arm the revised ones.

## Task prompt

> You are the fledge planner. Read and follow `<SKILL_ROOT>/agents/fledge-planner.md`, and write the
> plan in the shape of `<SKILL_ROOT>/references/templates/plan.md`.
>
> Phase `01-chain-owner-gate`. The ticket text below **is** the source-of-truth. Do not fetch it from
> Linear or anywhere else, do not call any MCP tool, and do not survey a real codebase — assume a
> mature Django monorepo with a `guest` app (`guest/models/reservation.py`,
> `guest/selectors/reservation.py`) and a `guest_journey` app that schedules and sends messages
> (`guest_journey/services/scheduling.py`, `guest_journey/services/sending.py`). Message scheduling
> is risk level 5.
>
> **MSG-6200 — Back-to-back: send Guest Journey messages once per chain**
>
> A back-to-back chain is two or more reservations for the same guest at the same hotel where each
> leg's departure date equals the next leg's arrival date. Today every leg gets every Guest Journey
> message. Change it so each message type goes out once per chain.
>
> - Add a nullable `b2b_owner` FK on `Reservation` pointing at the chain's owning leg (the first
>   leg). Recompute it whenever a reservation in the chain is created, updated or cancelled via PMS
>   sync.
> - Keep scheduling every message for every leg. At send time, a leg whose `b2b_owner` is another
>   reservation is suppressed for message types the owner sends.
> - Gate the behaviour per hotel with a `b2b_dedupe` feature flag.
> - Record each suppression so support can explain why a guest got one message, not two.
>
> Write the plan to `<OUT_DIR>/PLAN.md`. When done, reply with the path only.

## Setup / context

None beyond the prompt. The arm produces a plan, not code.

## Pass criteria (observable)

1. The plan contains an explicit enumeration (list or table) of the gate's **state changes**, each
   paired with an outcome — not a prose paragraph of "edge cases".
2. The enumeration includes **the owner has already sent a message and then becomes ineligible**
   (cancelled, dates moved, removed from the chain), with a stated outcome for the remaining legs.
3. The enumeration includes **the chain splits** and **the chain grows** (a leg added after
   messages were scheduled or sent), each with an outcome.
4. The plan lists every **new domain term** it introduces — field, model, flag and log/record
   names — with a one-line meaning (a glossary, or an equivalent table).
5. The plan records a **decision on the ticket's names** (`b2b_owner`, `b2b_dedupe`): kept with a
   reason or replaced, in the plan, before the migration landing unit — not carried over unexamined.

## Known failure modes (what the WITHOUT arm is expected to do)

- Mentions "edge cases: cancellation, PMS updates" in Approach or Risks without saying what
  happens to the other legs in each.
- Covers recompute-on-update but not the owner-already-sent leak: owner sends, then is cancelled,
  and the remaining legs stay suppressed for a message the guest will now never get for them.
- Uses `b2b_owner` and `b2b_dedupe` verbatim in the File plan and migration PR without a glossary
  or a naming decision.

## Notes

- Reproduces the back-to-back reservations project (retro 2026-10-14): the owner-already-sent leak
  (MSG-6397) and the `b2b_owner` → `back_to_back_reservation_owner` rename were both found in code
  review after other PRs were stacked on them.
- Both arms inherit the global `CLAUDE.md` and through it `~/.claude/shared/working-agreements.md`,
  which since 2026-10-14 carries *Settle state changes and names at design time*. The comparison is
  "the rule as a working agreement" vs "the rule as a plan section".
- Stochastic and an omission-shaped failure: a single WITHOUT run that happens to cover a case is
  not a pass unless the criterion holds as written.
