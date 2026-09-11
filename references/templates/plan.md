# Phase <id>: <title>

## Surface
<One of: `frontend` | `backend` | `full-stack` | `library-internal`. This drives the
test and QA approach downstream — see `references/qa-by-surface.md`. Justify in one line
from the File plan: e.g. "backend — only `backend/myapp/...` modules and an API contract,
no UI." Use `library-internal` for pure library/CLI/migration/internal-logic phases with
no UI and no externally-reachable service surface.>

## Source contract
- **Source-of-truth:** <SoT entry from manifest>
- **Requirements** (cite source line/section):
  - R1 — <requirement> (<source ref>)
  - R2 — ...

## Scope
- **In scope:** <bullets>
- **Out of scope:** <bullets — and where out-of-scope work lives>

## Bounded context
- **Domain / Django app this work belongs to:** <name>
- **Other apps this touches:** <list — and how: shared utility import, public service API, ORM relation, signal, etc. Cross-app imports are fine when they come from a shared location or follow established precedent.>
- **New coupling introduced:** <any new cross-app dependency that isn't using a shared location or following precedent — call it out explicitly. If you're moving away from a bad pattern, say so.>

## Approach
<2–5 paragraphs, staff-engineer level. What's the shape? What's the key invariant? What's the risky edge?>

## Reuse vs novelty
For each major piece of work, name an existing pattern/utility/component you considered:

| Piece | Existing pattern considered | Decision | Rationale |
|---|---|---|---|
| Token storage | `accounts.utils.SessionStore` | reuse | Same lifecycle, same threat model |
| Email validator | `notifications.email.validators.basic` | new | Existing one accepts `+` aliases; SoT R3 forbids them |

If you choose new over reuse, the rationale must cite a concrete defect or mismatch — not a stylistic preference.

## File plan
Every row names the landing unit (`## Landing plan` below) that carries it. A row that
belongs to two units is two rows.

| PR | Action | Path | Purpose |
|---|---|---|---|
| PR1 | create | backend/myapp/x/y.py | ... |
| PR2 | edit | frontend/src/components/Z.vue | ... |

## Test plan (TDD)
List the failing tests to write first. Group by file.

- `backend/myapp/x/tests/test_y.py`
  - `test_<behavior>` — fails until <implementation piece> lands
  - ...

## Landing plan
How this phase reaches `main`. **Sub-phases split the planning; this splits the shipping** — they
are not the same cut and often don't line up. One landing unit is fine; say so and why.

| PR | Contents | Depends on | Why its own PR |
|---|---|---|---|
| PR1 | Behavior-neutral move of the status vocabulary to the model layer | — | Pure-move diff; reviewable at a glance and unambiguous if it breaks something |
| PR2 | The selector, bounds, and exclusion logging | PR1 | The substantive change; reviewed on its own rather than buried behind a move |

Rules:
- **Default to more than one** when the phase has a behavior-neutral move, a migration, a
  backend/frontend seam, or a piece with no consumer yet. Each of those is an independent revert
  boundary, and a reviewer can hold one in their head.
- **A reason per row, and it has to be concrete** — reviewability, an independent revert boundary, a
  lint/migration rule, deploy ordering. "It's big" is not a reason; name what the split buys.
- **A piece with no caller in this phase is its own PR or is deferred.** It is the cheapest thing to
  drop and the most likely to be wrong (see checklist C12).
- **Size is a symptom, not the criterion.** A 900-line PR that is one coherent change is fine; a
  300-line PR mixing a move, a migration, and a feature is not.

## Sub-phases (if any)
### <id>.1 — <slug>
- **Scope:** <one paragraph>
- **Depends on:** <sibling ids or "none">
- **Source refs:** <which SoT sections drive this sub-phase>

## Risks
For each risk, name the mitigation, the rollback path, and whether a feature flag / dynamic-rollout gate is needed.

| Risk | Mitigation | Rollback path | Rollout gate |
|---|---|---|---|
| Concurrent signup race | `select_for_update` on email lookup | DB-level unique constraint protects integrity if check fails | none |
| OHIP delta storm on rollout | Behind dynamic rollout util | Disable flag, no data fix needed | dynamic_rollout key |

If the project `CLAUDE.md` flags any system as high-risk (e.g. message scheduling = risk level 5), and this phase touches it, mark "Risk level: 5 — extra caution required" at the top of this section.

## Open questions
(These must be resolved before implementation. If any exist after review, the checkpoint asks the user.)
