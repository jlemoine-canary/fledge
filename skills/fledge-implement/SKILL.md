---
name: fledge-implement
description: Use when the user says "implement <phase>", "make the tests pass", or after /fledge:fledge-test has written failing tests for a phase. Hands the reviewed plan and failing tests to the fledge-implementer subagent, who iterates TDD-style until tests pass and CLAUDE.md standards are met.
---

# fledge-implement

TDD green stage. Spawn `fledge-implementer` with the reviewed plan and failing tests. The implementer iterates until tests pass.

## Preconditions

- `REVIEW-PLAN-final.md` verdict = PASS
- `TESTS.md` exists and confirms tests are in red state
- User has approved the plan checkpoint (or this skill was invoked autonomously by the orchestrator)

## Arguments

- **Phase id** (e.g. `01-auth-refactor`). If omitted, use the most recent phase with a PASS plan review.
- **`--leaves-first`** — if the phase has sub-phases, implement leaves before parents (default)
- **`--here`** — implement just this phase, not its sub-phases

## Process

### 0. Worktree, branch & signing pre-flight

Before any code change:

1. **Worktree & branch check.** Prefer a dedicated git worktree over a shared checkout (e.g. the repo's main checkout) — concurrent agent sessions sharing one working tree + HEAD collide (another session's `checkout`/`commit`/`push` moves the branch ref and HEAD, reverts your files, or lands your commit on the wrong branch). Determine current vs default branch with `git rev-parse --abbrev-ref HEAD` and `git symbolic-ref refs/remotes/origin/HEAD`.
   - If not already in a dedicated worktree for this branch, create one off the up-to-date default: `git fetch origin <default>`, then `git worktree add <path> -b <git-user>/<TICKET-ID>/<phase-slug> origin/<default>`, and run from there. Confirm with `git worktree list` and that the branch's merge-base is a default-branch commit.
     - `git-user`: prefix of `git config user.email` before `@`
     - `TICKET-ID`: first `linear-issue` ref in `.fledge/SOURCES.md` (e.g. `STAY-2122`); if none, use the phase id (e.g. `01-auth-refactor`)
     - `phase-slug`: phase id with leading number stripped, kebab-case
     - Example branch: `jlemoine/STAY-2122/auth-refactor`
   - If the user explicitly wants to stack on another in-flight branch: base the worktree on that branch instead of the default, and say so.
   - If already on a suitable non-default branch in a dedicated worktree: stay put. Print `Continuing on branch <name>.`
   - Never disturb another worktree's/checkout's uncommitted files. If the only checkout is occupied by unrelated uncommitted work, ASK the user (don't stash silently).

2. **GPG signing check.**
   - `git config commit.gpgsign` must be `true`. If not, escalate — do not flip it for them.
   - `git config user.signingkey` must be set. If unset or set to a key not in `gpg --list-secret-keys`, escalate.
   - If signing is misconfigured, write `BLOCKED-gpg.md` and STOP. Do not commit unsigned. Do not skip with `--no-gpg-sign`.

These checks run once at the top of `/fledge:fledge-implement`. If the orchestrator already ran them, pass `--skip-preflight` to avoid re-checking.

### 1. Determine implementation order

Walk the phase tree. Find all leaves (phases with no sub-phases) — these are implemented first. Then their parents in dependency order. Then the root.

If `--here`, skip the tree walk.

### 2. For each phase to implement (in order)

#### a. Check dependencies
If the plan's `## Sub-phases` section lists `Depends on`, confirm those sub-phases are implemented before starting this one.

#### b. Spawn `fledge-implementer`

Pass a self-contained prompt:
- Phase directory
- Path to `PLAN.md`, `REVIEW-PLAN-final.md`, `TESTS.md`
- Source manifest path + SoT id
- Project `CLAUDE.md` path
- List of failing test files + test commands to run them
- Max iterations per test: 5 (then escalate with BLOCKED-*.md)

#### c. Handle the implementer's return

The implementer writes `IMPLEMENTATION.md` with:
- Files changed
- Deviations from plan (if any)
- Any `BLOCKED-*.md` files (if iteration cap hit)
- Any `DEVIATION.md` (if plan contradicted SoT)
- Any `TEST-DEFECT.md` (if a test seemed wrong)

**Handle escalations:**

- **BLOCKED** — write a summary, checkpoint to user: "Implementer got stuck on test X after 5 tries. Options: (a) review the test, (b) revise plan, (c) accept as known limitation."
- **DEVIATION** — the plan was wrong. Route back to `/fledge:fledge-review plan` with the deviation doc as input.
- **TEST-DEFECT** — route back to `/fledge:fledge-test` to fix the test, then re-spawn implementer.

### 3. Post-implementation sanity

After implementer returns success:
1. Run the full test suite — confirm no regressions in untouched areas
2. Run lint/typecheck per `CLAUDE.md` (`make check-fix`, `make typecheck-backend`, etc.)
3. If any fail, re-spawn implementer with the failures

### 4. Commit

Default to committing — fledge runs assume per-phase atomic commits unless the user disabled it.

- **Conventional Commits**: `feat(<scope>): <subject>`, `fix(<scope>): <subject>`, etc. Subject under 70 chars.
- **GPG signed** (verified in step 0).
- `Co-Authored-By` welcomed.
- Body cites the SoT reference (Linear ID, Notion link) when available.

Example:
```
feat(accounts): implement signup email uniqueness enforcement

Tests from TESTS.md pass. Concurrent-signup race covered by
select_for_update + DB-level unique constraint.

Closes STAY-2122.

Co-Authored-By: Claude <noreply@anthropic.com>
```

If the user opted out of mid-fledge commits, leave the changes uncommitted and note in `IMPLEMENTATION.md`.

### 5. Output

```
✓ Phase <id> implemented.
  Files changed: <N>
  Tests passing: <N>
  Non-blocking plan nits deferred: <N> (listed in IMPLEMENTATION.md)

Next: /fledge:fledge-review code <id>
```

## Context-budget management

Implementation can touch many files — keep the orchestrator context low by passing **paths** to the implementer, not file contents. The implementer has `Read` and will fetch what it needs.

If a phase has >10 files in its plan, consider splitting across two implementer spawns — by file group. The orchestrator does this split if the plan allows clean separation.

## What this skill does NOT do
- Write tests (that's `/fledge:fledge-test`)
- Review the implementation (that's `/fledge:fledge-review code`)
- Run QA — browser or API (that's `/fledge:fledge-qa`)

## Tools needed
- `Read`, `Write`, `Bash`, `Glob`
- `Agent` (spawn `fledge-implementer`)
- `AskUserQuestion` (for escalations)

## Related
- Subagent: `fledge-implementer`
- Next: `/fledge:fledge-review code`
- Back-steps: `/fledge:fledge-test` (test defect), `/fledge:fledge-review plan` (plan deviation)
