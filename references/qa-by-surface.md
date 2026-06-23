# QA by surface

Fledge does not assume every phase is a frontend phase. The QA stage (and the
test stage) adapt to the **surface** the phase actually touches. A backend-only
phase gets API/integration QA, not a browser; a pure-library phase may legitimately
have no QA layer above its tests.

This is the single source of truth for: how surface is classified, and what QA
each surface gets. `skills/fledge-qa/SKILL.md`, `agents/fledge-qa-engineer.md`
(frontend), and `agents/fledge-qa-engineer-backend.md` all defer to this file.

## The non-negotiable guardrail

**QA exercises the LOCAL dev environment only — never staging, never production.**
This holds for every surface. The browser QA hits `localhost:8080`; the backend QA
hits the local service (e.g. `localhost:8000`). "Hitting an endpoint" in fledge
always means a locally-running service. If a local service cannot be stood up,
**escalate** — do not point QA at a deployed environment.

## Surfaces

| Surface | What it means | QA agent | QA layer |
|---|---|---|---|
| `frontend` | User-facing UI (Vue/React/SPA), no meaningful new server contract | `fledge-qa-engineer` | Playwright browser flows |
| `backend` | API / service / domain logic / async jobs, no UI change | `fledge-qa-engineer-backend` | API + side-effect QA against the local service |
| `full-stack` | Both a UI change and a new/changed server contract | both agents | browser flows **and** API/side-effect QA |
| `library-internal` | Pure library, CLI, migration-only, or internal logic with **no** UI and **no** externally-reachable service surface | none | skip-with-rationale (the test stage is the coverage), or a CLI/data smoke if the phase ships a CLI |

## How surface is determined

Deterministic, in priority order:

1. **`PLAN.md` `## Surface` field.** The planner classifies the phase and records
   it. This is the authoritative signal — read it first.
2. **Fallback inference** (when the field is missing or a `--surface` override is
   not given): look at the plan's `File plan` table and, if implemented, the git
   diff:
   - paths under a frontend root only (`frontend/`, `*.vue`, `*.tsx`, `src/components/`) → `frontend`
   - paths under a backend root only (`backend/`, `*.py`, API/serializer/model/task modules) → `backend`
   - both → `full-stack`
   - none of the above reachable surfaces (only internal modules, CLI, migrations) → `library-internal`
3. **Explicit `--surface=<value>` arg** to `/fledge:fledge-qa` overrides everything
   (escape hatch for misclassification).

When inference is ambiguous (e.g. a backend change that the SoT says is consumed by
an existing UI), prefer the broader surface and say so in `QA.md`.

## Per-surface QA playbook

### frontend
Driven by `fledge-qa-engineer` (Playwright MCP). Golden path + key edge cases
(empty/error/permission-denied/concurrent) + regression guards on at-risk features.
See that agent for browser specifics. Requires the Playwright MCP (auth check).

### backend
Driven by `fledge-qa-engineer-backend`. The unit/integration tests already ran in
the test/implement stages; this layer is the **contract + side-effect smoke against
the running local service**:

1. **Stand up / confirm the local service** per `CLAUDE.md` (e.g. `cd backend && make up`,
   port 8000). Local only — see the guardrail above.
2. **Exercise the contract** using, in priority order:
   1. the project's own API/integration harness if one exists (e.g. pytest API tests
      with the project's test client) — extend it, don't reinvent;
   2. otherwise `httpx`/`curl` against the local service.
   Assert status codes, response shape/contract vs the SoT, error paths, and
   auth/permission behavior. Cover idempotency/concurrency where the SoT requires it.
3. **Verify side effects** — the things mocked-out unit tests can't catch: DB state
   (via the project's shell/ORM or a read endpoint), enqueued jobs/messages, emitted
   events/signals, and logs. Use read-only verification; don't mutate shared data
   destructively.
4. **Do not re-run unit tests** — that already happened. QA is the integration layer
   above them.

Does **not** require the Playwright MCP.

### full-stack
Run both playbooks. Typically: backend API/side-effect QA confirms the contract,
then browser QA confirms the UI consumes it correctly end-to-end.

### library-internal
There is no service or UI surface to exercise, so browser/API QA does not apply.
The test stage's unit/integration coverage **is** the QA. The QA engineer writes a
`QA.md` recording a **justified skip**: which tests provide the coverage, and why no
browser/API layer applies. If the phase ships a CLI, run a CLI smoke (invoke the
command, assert output/exit code) instead of skipping. Never invent browser scenarios
to have "done QA".

## Notes
- A phase that the planner marked `backend` but whose SoT requirements are entirely
  user-facing is a classification smell — flag it rather than silently skipping the UI.
- Surface is per-phase. A full-stack project split into a `backend` sub-phase and a
  `frontend` sub-phase gets the right QA for each leaf.
