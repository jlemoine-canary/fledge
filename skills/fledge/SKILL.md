---
name: fledge
description: Top-level orchestrator for the full project lifecycle. Runs auth-check → ingest → plan → review-plan → test → implement → review-code → qa with two user checkpoints (after final plan review, after QA). Use when the user says "fledge this project", "run the full pipeline", or provides a batch of source documents and wants everything done.
version: 0.1.0
---

# fledge (orchestrator)

Run the full pipeline from source documents to shipped (QA-passed) code. Two user checkpoints:
1. After the final plan review passes
2. After QA passes

All other stages run autonomously (subject to context-budget escalations).

## When to run
- User provides a batch of source documents and says "run the pipeline"
- User has an existing `.fledge/` project and wants to proceed through all remaining stages
- Starting fresh from a single source doc (will prompt to ingest)

## When NOT to run
- User only wants one stage — they should invoke that stage directly (`/fledge-plan`, `/fledge-review code`, etc.)
- Project has active unresolved escalations — resolve first, then resume

## Arguments

- `--from=<stage>` — resume from a specific stage (e.g. `--from=implement` assumes plan + review passed)
- `--phase=<id>` — operate on a specific phase (default: the top-level phase in `.fledge/phases/`)
- `--deep` — allow sub-phase depth > 3 (propagates to `/fledge-plan`)
- `--dry-run` — print the plan of execution without running subagents

## Process

### 0. Context budget — pre-flight
Before starting, print a scope estimate:
- Number of sources to ingest
- Rough size budget
- Expected number of subagent spawns across the pipeline

If estimated total work is large (>20 subagent spawns), warn:
> ⚠ This pipeline will spawn ~N subagents across ~M stages. Estimated duration: X-Y minutes.
> Consider running individual stages manually to maintain context headroom.

### 1. Auth check
Invoke `/fledge-auth`. If any connector fails, STOP with instructions.

### 1.5. Worktree, branch & GPG pre-flight

Before any code-touching work:

- **Isolated worktree.** Don't work in a checkout that other sessions may also be using (e.g. the repo's main checkout). Concurrent agents sharing one working tree + HEAD collide: another session's `checkout`/`commit`/`push` moves the branch ref and HEAD out from under you, reverts your files, or lands your commit on the wrong branch. Determine the default branch with `git symbolic-ref refs/remotes/origin/HEAD`.
  - If the current directory is already a dedicated worktree for this project's branch (not the shared main checkout, no other session using it), stay put.
  - Otherwise create one off the up-to-date default branch: `git fetch origin <default>`, then `git worktree add <path> -b <git-user>/<TICKET-ID>/<phase-slug> origin/<default>`, and run all later stages there. (TICKET-ID = first Linear source in SOURCES.md, else phase id.) Confirm with `git worktree list` and that the new branch's merge-base is a default-branch commit.
  - Branch off the default branch **unless** the user explicitly wants to stack on another in-flight branch — then base the worktree on that branch and say so.
  - Never disturb another worktree's/checkout's uncommitted files. If the only available checkout is occupied by unrelated uncommitted work, ASK before doing anything.
- Verify `git config commit.gpgsign` is `true` AND `git config user.signingkey` resolves to a key in `gpg --list-secret-keys`. If signing is misconfigured, STOP and ask the user to fix it before proceeding.

These also run inside `/fledge-implement`; the orchestrator passes `--skip-preflight` to avoid re-checking.

### 2. Ingest (if needed)
If `.fledge/SOURCES.md` doesn't exist:
- Prompt user for source docs (or parse from the invocation message)
- Invoke `/fledge-ingest`

If it exists, verify freshness (SoT last-updated vs current) and offer to refresh.

### 3. Plan
Invoke `/fledge-plan` for the target phase (creates tree if sub-phases).

### 4. Review plan
Invoke `/fledge-review plan --sub-phases` (the sub-skill returns a verdict; the orchestrator owns the checkpoint).

After all plans return PASS (or ESCALATE):

### 5. 🛑 Checkpoint 1 — "Plan approved?"

Apply `references/checkpoint-protocol.md`:
1. Write `.fledge/checkpoints/<timestamp>-plan.md` with:
   - Plan tree (all PLAN.md files)
   - SoT and manifest summary
   - Non-blocking nits across all plan reviews
   - Count of sub-phases and estimated implementation scope
2. `AskUserQuestion`:
   - `approve` — continue
   - `iterate` — re-run `/fledge-review plan` with user feedback
   - `abort` — stop; write resume file
   - `details` — print the plan tree inline, then re-prompt

On abort: `.fledge/RESUME.md` is written with exact `--from=plan` recovery command.

### 6. Test → Implement → Review code (looped, per phase leaf-first)

For each phase in leaves-first order:
1. `/fledge-test <phase>` — write failing tests
2. `/fledge-implement <phase>` — make them pass
3. `/fledge-review code <phase>` — 3-round review (returns a verdict)
4. If review returns BLOCK, the review skill handles the re-implement loop internally. If it ESCALATEs, surface to user immediately (do not proceed to next phase).

**Context budget check** between phases. At 50%, warn. At 70%, checkpoint the user.

### 7. QA
Invoke `/fledge-qa` once, after all phases are implemented and reviewed. The QA engineer exercises the whole change set end-to-end. The skill returns a verdict; the orchestrator owns the checkpoint.

If QA fails, the skill routes back to `/fledge-implement` → `/fledge-review code` → `/fledge-qa` internally, up to 3 QA rounds.

### 8. 🛑 Checkpoint 2 — "Ship it?"

Write `.fledge/checkpoints/<timestamp>-done.md` with:
- All phases and their statuses
- All passing tests and QA scenarios
- Combined list of non-blocking nits
- Suggested next actions (PR, deploy) — **described, not taken**

`AskUserQuestion`:
- `approve` — declare fledged; summarize
- `iterate` — take specific feedback, route to the right stage
- `abort` — leave state as-is

## Failure handling

If any stage autonomously escalates (context at 70%, review cap hit, TDD stuck, QA stuck), **stop immediately** — treat as checkpoint. Do not retry blindly.

The escalation checkpoint has 3 options:
- `continue` with user's specific guidance
- `restart-stage` from a clean slate
- `abort` and save resume file

## Tools needed
- `Read`, `Write`, `Bash`, `Glob`
- `Agent` (many — it orchestrates every subagent-spawning sub-skill)
- `AskUserQuestion` (checkpoints + escalations)
- `Skill` (to invoke `/fledge-auth`, `/fledge-ingest`, `/fledge-plan`, etc.)

## Design principles

- **Do NOT re-implement what sub-skills do.** Invoke them. This is pure orchestration.
- **Do NOT paste source content between sub-skills.** They re-fetch live from the manifest references. This keeps orchestrator context small.
- **Do NOT skip checkpoints.** Users approved this design explicitly.
- **Do NOT add stages.** If a user wants a new stage (e.g. "security review"), that's a new skill that composes into the pipeline — not a patch to this orchestrator.

## Related
- Sub-skills: `/fledge-auth`, `/fledge-ingest` (with `--append`), `/fledge-plan`, `/fledge-review` (plan|code), `/fledge-test`, `/fledge-implement`, `/fledge-qa`
- References: `checkpoint-protocol.md`, `context-budget.md`, `severity-rubric.md`, `source-manifest-format.md`, `sot-snapshot.md`, `subphase-depth.md`, `templates/`
