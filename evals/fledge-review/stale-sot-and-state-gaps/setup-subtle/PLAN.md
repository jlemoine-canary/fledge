# Phase 01: Back-to-back owner gate

## Surface
backend — `guest` and `guest_journey` modules only, no UI.

## Source contract
- **Source-of-truth:** S1 — Back-to-back reservation design (Notion)
- **Requirements:**
  - R1 — Add nullable `Reservation.b2b_owner` FK, set on the second leg of a pair (S1 § Design)
  - R2 — At scheduling time, skip pre-arrival and check-in messages for a reservation with a `b2b_owner` (S1 § Design)
  - R3 — Recompute `b2b_owner` on PMS reservation update (S1 § Design)

## Scope
- **In scope:** pairs; pre-arrival and check-in messages; owner recompute on PMS update.
- **Out of scope:** chains of three or more legs (S1 § Scope); other message types.

## Bounded context
- **Domain / Django app this work belongs to:** `guest_journey`
- **Other apps this touches:** `guest` — model field on `Reservation`, selector for the partner lookup.
- **New coupling introduced:** none; `guest_journey` already reads `guest` models.

## Approach
Add `b2b_owner` to `Reservation`. A selector, `ReservationSelector.get_b2b_partner(reservation)`,
finds the previous consecutive reservation for the same guest and hotel. The PMS sync handler
calls `recompute_b2b_owner(reservation)` after saving a reservation, setting or clearing
`b2b_owner` on it and on its partner.

`guest_journey.services.scheduling.schedule_messages` skips pre-arrival and check-in specs when
`reservation.b2b_owner_id` is set. Everything else is scheduled as today.

Gated per hotel by a `b2b_dedupe` feature flag; with the flag off, `b2b_owner` is still written but
scheduling ignores it.

## Reuse vs novelty
| Piece | Existing pattern considered | Decision | Rationale |
|---|---|---|---|
| Partner lookup | `ReservationSelector` | reuse | Same model, same query shape |
| Flag | `Features.is_enabled(hotel, ...)` | reuse | Standard per-hotel flag |

## File plan
| PR | Action | Path | Purpose |
|---|---|---|---|
| PR1 | edit | guest/models/reservation.py | Add `b2b_owner` FK |
| PR1 | create | guest/migrations/0412_reservation_b2b_owner.py | Migration |
| PR2 | edit | guest/selectors/reservation.py | `get_b2b_partner` |
| PR2 | edit | guest/services/pms_sync.py | Call `recompute_b2b_owner` |
| PR3 | edit | guest_journey/services/scheduling.py | Skip specs when `b2b_owner_id` is set |

## Test plan (TDD)
- `guest/tests/test_selectors.py`
  - `test_get_b2b_partner_returns_previous_consecutive_reservation`
  - `test_get_b2b_partner_none_when_gap_between_stays`
- `guest/tests/test_pms_sync.py`
  - `test_recompute_sets_b2b_owner_on_second_leg`
  - `test_recompute_clears_b2b_owner_when_dates_move_apart`
- `guest_journey/tests/test_scheduling.py`
  - `test_skips_pre_arrival_and_check_in_when_b2b_owner_set`
  - `test_schedules_normally_when_flag_off`

## Landing plan
| PR | Contents | Depends on | Why its own PR |
|---|---|---|---|
| PR1 | `b2b_owner` field + migration | — | Migration ships alone per repo rule |
| PR2 | Selector + recompute on PMS sync | PR1 | Writes the field; no behavior change while nothing reads it |
| PR3 | Scheduling skip behind `b2b_dedupe` | PR2 | The behavior change; own revert boundary |

## Sub-phases
None.

## Risks
Risk level: 5 — extra caution required

| Risk | Mitigation | Rollback path | Rollout gate |
|---|---|---|---|
| Guest gets no pre-arrival message | Only skip when partner exists | Disable `b2b_dedupe` | `b2b_dedupe` |
| Recompute load on PMS sync | One indexed query per save | Disable flag | `b2b_dedupe` |

## Open questions
None.
