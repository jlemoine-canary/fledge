# Changelog

All notable changes to the fledge plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.0] - 2026-09-11

**Verify this release by:** the next `PLAN.md` fledge produces containing a `## Landing plan` table
whose rows are echoed in the `File plan`'s new `PR` column — and, when that plan has more than one
row, `/fledge:fledge-implement` opening one branch per row instead of one per phase.

### Added

- **`## Landing plan` in the plan template** (`references/templates/plan.md`) — how the phase reaches
  `main`, as a table of `PR | Contents | Depends on | Why its own PR`. Distinct from `## Sub-phases`:
  sub-phases cut the *planning*, the landing plan cuts the *shipping*, and the two rarely coincide.
  Carries four rules: default to more than one unit when there's a behavior-neutral move, a migration,
  a backend/frontend seam, or a piece with no consumer; a concrete reason per row; a caller-less piece
  is its own PR or is deferred; and **size is a symptom, not the criterion** — a coherent 900-line PR
  is fine, a 300-line PR mixing a move, a migration and a feature is not.
- **`PR` column on the `File plan` table.** Every row names the unit that carries it; a row belonging
  to two units is two rows. This is the table `/fledge:fledge-implement` reads, so without it a good
  split never reaches the branch.
- **`--pr=<id>` on `/fledge:fledge-implement`**, and a new step 0 that reads the landing plan before
  any code change: one row → one branch as before; more than one → one row at a time in dependency
  order, each on its own branch, taking only the `File plan` rows tagged with it. Plans predating the
  section are treated as one unit and say so in `IMPLEMENTATION.md` rather than inventing a split the
  reviewer never saw.
- **`P11. Landing plan`** in `references/code-review-checklist.md` — review the split as a claim.
  A single unit needs a reason better than "it's all one feature"; every `File plan` row names a unit
  and no row appears twice; and units that must land together within the hour are one PR with extra
  ceremony, not three.
- **Eval fixture `evals/fledge-plan/pr-slice-declaration/`** with both arms recorded.

### Changed

- `agents/fledge-planner.md` step 7 — fill the landing plan deliberately; it is the one section with
  no natural default. Ask how many times this phase should reach `main`, not how many pieces of work
  it contains.
- `skills/fledge-plan/SKILL.md` — new "Landing plan vs. sub-phases" section naming the two cuts.
- `skills/fledge-implement/SKILL.md` — branch slug is the landing-unit slug when the plan has more
  than one row; output reports which unit landed and what remains. The context-budget note now says a
  >10-file split across implementer spawns is a split *within* one PR — PR boundaries are the plan's
  to set, not the orchestrator's.
- Reviewer checklist indexes → `P1–P11`.

### Evidence

RED, from `evals/fledge-plan/pr-slice-declaration/result-2026-09-11-baseline.md`: a baseline planner
given the real CC-3083 ticket scored **2 of 5** criteria. It did split — into `## Sub-phases` with
dependencies — but only the smallest unit mentioned a PR, the unit corresponding to the real
+1610-line PR (#53519, open 24 days) carried no PR-boundary reasoning at all, and the `File plan` was
one undifferentiated table with two sub-phases collapsed into a single cell. Across all of 0.5.1,
grep for slice / shippable / "one PR per" / "separate PRs" returned **zero matches**, and
`fledge-implement` created exactly one branch per phase — so one PR per phase was structural, not a
judgment lapse.

Corroborated by the four real `PLAN.md` artifacts on disk: none has a landing section. The one that
did work out a PR split (`01-span-date-proxy`) did so only because a project lint rule forced it, and
wrote the answer into `PRECONDITION-PRS.md` — an ad-hoc file no skill defines and no reviewer knows
to check.

GREEN: **5 of 5**, with the arm using the template's exact section name, column headers and rule text.
One validity caveat is recorded in the result file — the arms differed in Linear access, which is not
what flipped criteria 1/3/5 but was uncontrolled; the fixture is now hermetic.

## [0.5.1] - 2026-09-11

**Verify this release by:** the next release cut with `/fledge:fledge-writing-skills` producing a
checklist that continues past "merged" — i.e. it tells you to run `claude plugin marketplace update`
alongside `claude plugin update`, to check `installed_plugins.json` rather than the command's
success message, and to name a per-release acceptance artifact in the changelog entry. (This entry
is itself an instance of that last rule.)

### Changed

- **The release checklist now ends at "verified in a real run", not at "merged"**
  (`skills/fledge-writing-skills/SKILL.md` § Releasing a skill change, and `README.md`
  § Releasing changes). Both previously stopped at the version bump and the changelog entry, with
  the post-merge install as a trailing aside. They now carry six ordered steps — three before the
  merge, three after — and state plainly that merged work sits inert in a version-gated cache until
  the cache moves and something exercises it.
  - **`claude plugin marketplace update fledge` is now named explicitly**, alongside
    `claude plugin update fledge@fledge`. The directory marketplace caches the plugin's advertised
    version; without the refresh the listing still names the old one, so the update no-ops *and
    reports success*. The old README named only the second command — which is precisely how
    0.2.1, 0.3.0 and 0.4.0 went stale.
  - **Verification is now defined against the registry, not the success message**: the new version
    directory must exist under `~/.claude/plugins/cache/fledge/fledge/`, `installed_plugins.json`
    must show that version with `gitCommitSha` equal to the merge commit, and the cache's `skills/`
    and `references/` must diff clean against the checkout.
  - **Every release must now name its own acceptance test.** The changelog entry declares one
    artifact or observable behavior only that version can produce; step 6 is running the pipeline
    for real and confirming it appears. This makes "did it ship?" answerable, and catches a change
    that installs correctly but is inert or wrong.
- **Four new entries in the "Rationalizations to reject" table**: "it's merged, so it's shipped",
  "I'll update the plugin next time I use fledge", "`claude plugin update` said it succeeded", and
  "the diff is obviously correct, a real run is overkill".

### Context

The 2026-09-11 fledge program retro found the installed cache pinned at 0.2.0 since June while the
repo was at 0.4.0 — 993 references to the 0.2.0 cache path across session transcripts and no other
version. Versions 0.2.1, 0.3.0 and 0.4.0 were each authored, reviewed, merged and changelogged, and
**never executed once**: the trigger-first descriptions, the TDD rationalization guards,
`fledge-writing-skills` itself, `fledge-eval`, and the entire deterministic-handoff redesign
produced no effect on any ticket for three months, with nothing in the process noticing. 0.5.0 was
the first release to reach the cache on its merge day. This change makes that the default rather
than something the user has to remember.

## [0.5.0] - 2026-09-11

Driven by the first fledge program retro
(`~/.claude/shared/retros/2026-09-11-fledge-program-retro.md`), which reviewed 98 non-author
review comments across the 87 PRs authored since 2026-06-01 and the four `.fledge/` projects on
disk. Every item below is traced to review feedback fledge's own three-round review did not catch.

### Added

- **`references/code-review-checklist.md` — four new code-mode items and one new plan-mode item**,
  covering the classes that reached human reviewers on fledge-built PRs:
  - **C10 Comment altitude** — comments that restate the diff, narrate the change, or carry
    PR-description-length rationale. The single most-repeated style complaint across reviewers
    (raised on four separate PRs by three people).
  - **C11 User-facing copy follows behaviour** — when a matching rule or condition changes, grep
    the feature's tooltips, labels, i18n keys, `aria-label`s, and docstrings. A shipped tooltip
    read "Automatically linked via matching phone number" after the match moved to email.
  - **C12 YAGNI — machinery without a caller** — does the new abstraction/cache/memo/knob have a
    caller in this diff? Memoization landed with "no consumers yet" and had to be walked back.
    Also covers over-large functions, unvalidated micro-optimizations, and duplicate guards.
  - **C13 Write-endpoint idempotency** — every new side-effecting `POST`/`PUT` needs a
    double-click answer; dedup locks must be acquired *after* validation and released on every
    failure path.
  - **P10 Data-migration completeness** — for any new column/filter/index: who backfills the
    existing rows, which *other* query's plan just changed, and which sibling surfaces share the
    model. Also: a remediation's `UPDATE` must re-apply the predicates its `SELECT` used.

### Changed

- **C1 extended — "prove the guarantee, not the framework."** The mutation-test rule was stated
  generically and missed the same three shapes repeatedly: database-level defaults (a test using
  `objects.create()` passes with `db_default` deleted), CLI flag matrices, and arguments
  advertised as safety boundaries (assert *isolation*, and assert *which* error). Six of the
  seven review findings in this class came from the review bot, not from fledge.
- **P1 sharpened** — names "add a new X" vs "change the existing X" as the highest-cost silent
  source-of-truth resolution. Both readings compile; the miss surfaced three weeks later as a
  revert.
- **C9** — new write endpoints get one extra pass for reuse of a read-scoped auth/permission class.
- `agents/fledge-reviewer-adversarial.md`, `agents/fledge-reviewer-constructive.md` — checklist
  indexes updated to P1–P10 / C1–C13.

### Removed

- **`references/subphase-depth.md`** and the `/fledge-plan --deep` flag. Zero of the four real
  fledge projects produced more than one phase, so the depth-3 cap, the recursion past level 2,
  and the escape hatch were never exercised — and the doc's own advice ("flat is better than
  nested") argued against them. Replaced by a fixed rule inline in `skills/fledge-plan/SKILL.md`:
  one level of sub-phases, then siblings; when in doubt, sibling. The numbering convention and the
  sub-phase-vs-sibling decision rule moved there intact.
- Consequently trimmed: the `--deep` argument in `skills/fledge/SKILL.md`, the depth-level and
  depth-cap plumbing in `skills/fledge-plan/SKILL.md` and `agents/fledge-planner.md`, and the
  "Depth justification" line in `references/templates/plan.md`.

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
