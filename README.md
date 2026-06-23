# fledge

A composable plugin for taking a project from source documents to shipped code.

Each stage is its own skill — run them individually or let `/fledge` orchestrate the full pipeline.

## Pipeline

```
auth → ingest → plan → review(plan) → test → implement → review(code) → qa
                          (checkpoint)                              (checkpoint)
```

Both checkpoints are owned by the orchestrator. Sub-skills return verdicts.

## Skills

| Skill | Purpose |
|---|---|
| `/fledge` | Top-level orchestrator — runs the full pipeline with checkpoints |
| `/fledge-auth` | Verify MCP connections (Notion, Linear, GitHub, Figma, Playwright) |
| `/fledge-ingest` | Fetch and index source docs; set source-of-truth. `--append` to add to an existing project. |
| `/fledge-plan` | Staff-engineer plan for a phase (recurses into sub-phases, max depth 3) |
| `/fledge-review` | Three-round review on a plan or implementation. Pass mode `plan` or `code`. |
| `/fledge-test` | Write failing tests (TDD red) |
| `/fledge-implement` | Senior-engineer TDD iteration until tests pass |
| `/fledge-qa` | Surface-aware QA — browser flows for UI, API + side-effect checks for backend, justified skip for pure library/CLI. Iterates until the implementation matches requirements. |

## Subagent personas

- `fledge-planner` — staff-engineer planner
- `fledge-implementer` — senior-engineer implementer (TDD)
- `fledge-reviewer-constructive` — finds issues with a collaborative lens
- `fledge-reviewer-adversarial` — actively tries to break the plan/code
- `fledge-reviewer-integrator` — reconciles prior two reviews, produces final verdict
- `fledge-qa-engineer` — frontend/browser QA (Playwright)
- `fledge-qa-engineer-backend` — backend QA: API + side-effect checks against the local service (no browser)

## Shared references

- `references/severity-rubric.md` — critical/major/minor/nit + consequential y/n rubric
- `references/context-budget.md` — 50% warn / 70% stop protocol
- `references/source-manifest-format.md` — `.fledge/SOURCES.md` format
- `references/sot-snapshot.md` — one fetch per review cycle, all personas read the snapshot
- `references/checkpoint-protocol.md` — orchestrator-owned checkpoint mechanics
- `references/subphase-depth.md` — depth cap and escape hatch
- `references/qa-by-surface.md` — surface taxonomy (frontend/backend/full-stack/library-internal) and the QA each gets
- `references/templates/` — output templates for plan, review, implementation, qa, tests

## State

Fledge writes state into `.fledge/` at the project root:

```
.fledge/
├── SOURCES.md
├── phases/
│   ├── 01-<slug>/
│   │   ├── PLAN.md
│   │   ├── TESTS.md
│   │   ├── IMPLEMENTATION.md
│   │   ├── REVIEW-PLAN-final.md
│   │   ├── REVIEW-CODE-final.md
│   │   ├── QA.md
│   │   ├── .sot-snapshot.md          # refreshed per review cycle
│   │   ├── review-history.md
│   │   ├── review-history/cycle-N/   # archived rounds
│   │   └── sub-phases/
│   │       └── 01.1-<slug>/...
│   └── ...
└── checkpoints/
    └── <timestamp>-<stage>.md
```

## Releasing changes

The installed plugin is a **version-gated cache** (at `~/.claude/plugins/cache/fledge/...`) — `claude plugin update` only picks up changes when the declared version increases. So every content change merged to `main` must:

1. Bump the version in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (they must stay in sync).
2. Add an entry to `CHANGELOG.md`.

Otherwise installed copies silently stay stale. After merging, pull the source checkout and run:

```
claude plugin update fledge@fledge
```

Individual `SKILL.md` files intentionally carry no `version:` frontmatter — the plugin version is the single source of truth for releases.
