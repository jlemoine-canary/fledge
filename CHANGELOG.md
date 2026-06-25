# Changelog

All notable changes to the fledge plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-06-25

### Added

- **Deterministic subagent handoff packages** (closes #3). The pipeline now hands
  off between stages via generated, reproducible packages instead of each subagent
  reconstructing its own view — Claude-Code-native (a pinned recipe, no scripts, no
  new runtime dependency):
  - `references/review-package-format.md` — single source of truth for the
    code-mode change set. Pins deterministic base/head resolution (recorded
    `.base-commit`, else `git merge-base`), the fixed git recipe, and the
    `REVIEW-PACKAGE.md` (+ sibling `REVIEW-PACKAGE.patch`) output. No editorial
    prose in the package — reviewers form the opinions. Supersedes the ad-hoc
    `CHANGES.md`.
  - `references/task-brief-format.md` — single source of truth for the implementer
    hand-off. Pins how `TASK-BRIEF.md` is assembled from `PLAN.md` + `TESTS.md` +
    `REVIEW-PLAN-final.md` (references not pasted bodies, verbatim test command,
    review residue as required-fixes vs deferred-nits).
  - Two eval fixtures documenting the baseline non-determinism the recipes fix:
    `evals/fledge-review/review-package-determinism`,
    `evals/fledge-implement/task-brief-minimal`.

### Changed

- `skills/fledge-review/SKILL.md` (step 2) now compiles the deterministic
  `REVIEW-PACKAGE.md` per the new reference; reviewers' `code`-mode artifact path is
  that file.
- `skills/fledge-implement/SKILL.md` records the phase `.base-commit` at branch
  creation and generates `TASK-BRIEF.md` for the implementer.
- `agents/fledge-implementer.md`, `fledge-reviewer-constructive.md`,
  `fledge-reviewer-integrator.md` — `Inputs` updated to name the standardized
  hand-off artifacts.

## [0.3.0] - 2026-06-25

### Added

- **Self-maintenance loop for fledge's own skills** (closes #2). Fledge can now
  author and validate its skills with evidence instead of vibes:
  - `skills/fledge-writing-skills/SKILL.md` — meta-skill encoding the
    RED→GREEN→REFACTOR authoring loop (no skill without a documented baseline
    failure first), the fledge SKILL.md/agent/reference conventions, the
    trigger-first ("Use when …") description rule, the release checklist, and a
    "Rationalizations to reject" section.
  - `skills/fledge-eval/SKILL.md` — lightweight, **Claude-Code-native** eval
    harness: runs a fixture scenario through a without-skill subagent and a
    with-skill subagent and reports the behavioral delta. No shell/Python, no new
    runtime dependency.
  - `references/skill-eval-protocol.md` — single source of truth for the eval
    mechanics (the controlled A/B, fixture anatomy, how to read the outcome).
  - `evals/` — in-repo behavioral fixtures: a README, scenario/result templates
    under `evals/_template/`, and two seed fixtures (`fledge-test/private-function-test`,
    `fledge-writing-skills/no-baseline-failure`).

## [0.2.1] - 2026-06-25

### Changed

- **Skill descriptions rewritten as triggers, not summaries.** All eight
  `skills/*/SKILL.md` descriptions now lead with their "Use when …" triggering
  conditions, with workflow detail trimmed to a short disambiguating clause.
  Improves auto-triggering accuracy (the agent routes on the description before
  it reads the body). Inspired by the convention in obra/superpowers.

### Added

- **Rationalization guards in the TDD discipline.** New "Rationalizations to
  reject" sections in `skills/fledge-test/SKILL.md` and
  `agents/fledge-implementer.md` enumerate and rebut the excuses an agent invents
  to skip the red stage, keep untested code, weaken assertions, grind past the
  iteration cap, or bypass GPG signing — hardening discipline under time pressure.

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
