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
- **Depth justification** (only if --deep): <why this needs level 4+>

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
| Action | Path | Purpose |
|---|---|---|
| create | backend/myapp/x/y.py | ... |
| edit | frontend/src/components/Z.vue | ... |

## Test plan (TDD)
List the failing tests to write first. Group by file.

- `backend/myapp/x/tests/test_y.py`
  - `test_<behavior>` — fails until <implementation piece> lands
  - ...

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
