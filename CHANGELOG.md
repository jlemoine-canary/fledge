# Changelog

All notable changes to the fledge plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-06-22

### Added

- **Surface-aware QA.** The QA stage no longer defaults to Playwright for every
  phase. Each phase is classified as `frontend`, `backend`, `full-stack`, or
  `library-internal`, and QA routes accordingly:
  - `references/qa-by-surface.md` — new shared playbook defining the taxonomy,
    the per-surface QA approach, and the **local-dev-only** guardrail (never
    staging/production).
  - `agents/fledge-qa-engineer-backend.md` — new backend QA agent: exercises the
    running local service's API contract and side effects (DB, jobs, events,
    logs) with no browser tools.
  - `## Surface` field added to the plan template; `fledge-planner` now classifies
    each phase, and `fledge-test` matches the TDD layer to the surface.

### Changed

- `fledge-qa` classifies the phase surface and spawns the matching agent(s);
  probes only the dev server(s) the surface needs.
- `fledge-qa-engineer` reframed as the explicit frontend/browser QA agent.
- `fledge-auth` treats the Playwright MCP as a **conditional, non-blocking**
  check — required only for frontend/full-stack QA, so backend-only pipelines no
  longer stop on a missing Playwright install.
- `qa.md` template carries `Surface` / `QA type` and a per-row surface column.

## [0.1.1] - 2026-06-12

### Added

- Forbid tests that call private functions directly — new anti-pattern in
  `skills/fledge-test/SKILL.md` and item C1 in
  `references/code-review-checklist.md`.

### Removed

- Per-skill `version:` frontmatter in `skills/*/SKILL.md` — the plugin version
  in `.claude-plugin/plugin.json` is the single release unit.

## [0.1.0] - 2026-06-08

### Added

- Initial plugin: lifecycle skills (`fledge`, `fledge-auth`, `fledge-ingest`,
  `fledge-plan`, `fledge-review`, `fledge-test`, `fledge-implement`,
  `fledge-qa`), subagent personas, shared references and templates.
- `marketplace.json` so the repo is installable as a local directory
  marketplace.
