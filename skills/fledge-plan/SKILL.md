---
name: fledge-plan
description: Use when the user says "plan this phase", "plan <phase-id>", "break this down", or after /fledge:fledge-ingest establishes sources. Produces a staff-engineer-level PLAN.md via the fledge-planner subagent, optionally splitting the phase into one level of sub-phases.
---

# fledge-plan

Produce `PLAN.md` for a phase. Spawns `fledge-planner` subagent. If the planner identifies sub-phases, this skill spawns one planner per sub-phase.

## Preconditions

- `.fledge/SOURCES.md` exists and has exactly one source-of-truth
- Project root has a `CLAUDE.md` (or the user has confirmed it has none — rare)

## Arguments

- **Phase identifier** — e.g. `01-auth-refactor`. If omitted, derive from the source-of-truth title and prompt user to confirm.

## Landing plan vs. sub-phases

Two different cuts, and conflating them is why phases ship as one oversized PR:

- **Sub-phases** divide the *planning and implementation* work. They are about scope and dependency.
- **The landing plan** divides how the phase reaches `main`. It is about review and revert boundaries.

A phase with no sub-phases can still land as three PRs; a phase with three sub-phases can land as one.
The planner fills `## Landing plan` in `PLAN.md`; `/fledge:fledge-implement` branches per row in it.
Format and the slicing rules live in `references/templates/plan.md`.

## Nesting: one level, then siblings

A phase may have sub-phases. A sub-phase may not. Past one level the plan stops being readable by a human at the final checkpoint, and every level adds a subagent hop and its handoff risk.

**Numbering**: top-level `01`, `02`, …; sub-phases `01.1`, `01.2`, …. Directory slug is `<number>-<kebab-slug>` (`01-auth-refactor`, `01.1-token-storage`).

**Sub-phase or sibling?** Make it a **sub-phase** only when it shares context or invariants with its parent that would be awkward to duplicate, *and* it can't start until the parent's scaffolding exists. Make it a **sibling** when it's independently reviewable and testable, or could ship on its own. **When in doubt, make it a sibling** — flat beats nested here.

If the split genuinely wants a third level, the top-level scope was wrong: re-split into siblings rather than nesting deeper.

## Process

### 1. Resolve the phase
- If no existing `.fledge/phases/<id>/` directory, create it and this is a top-level phase.
- If spawned for a sub-phase, parent passes the sub-phase id.

### 2. Check source-of-truth freshness
The SoT may have changed since ingest. Re-fetch it and compare last-updated timestamp against `.fledge/SOURCES.md`'s `Last updated`. If changed:
- Warn the user
- Offer: "Update SOURCES.md before planning? (recommended yes)"

### 3. Spawn `fledge-planner`
Pass in a self-contained prompt with:
- Phase id, phase directory path, and whether this is a top-level phase or a sub-phase
- Path to `.fledge/SOURCES.md`, SoT id
- Path to project `CLAUDE.md`
- Path to parent plan (if sub-phase)
- Paths to sibling phase plans (if any, for invariant context)
- The relevant `references/` docs (severity, source-manifest)

**Do NOT paste source-doc content** into the prompt. Pass the reference; the planner fetches live.

### 4. Handle the planner's output
The planner writes `PLAN.md` and returns either:
- A simple success
- A refusal (the phase wants a third nesting level, ambiguity in SoT)

### 5. Sub-phases

If `PLAN.md` contains a `## Sub-phases` section:
1. For each sub-phase entry, create a sub-phase directory
2. Spawn one `fledge-planner` per sub-phase — **sequentially** unless they're genuinely independent (check dependencies in the plan)
3. If 3+ sub-phases are independent, run them in parallel (single message, multiple Agent calls) to save wall time
4. Sub-phases do not nest further. If a sub-phase's plan asks for its own sub-phases, that's a re-split signal — stop and take it back to the user

**Context budget check** after each returning subagent. If at 50%, warn; at 70%, stop and checkpoint (see `../../references/context-budget.md`).

### 6. Verify the plan tree
After all sub-planners return, re-read the top-level plan and confirm:
- Every sub-phase in the parent's `## Sub-phases` list got a PLAN.md
- No orphan phase directories

### 7. Hand off
Output:
```
✓ Plans written:
  .fledge/phases/01-auth-refactor/PLAN.md
  .fledge/phases/01-auth-refactor/sub-phases/01.1-token-storage/PLAN.md
  .fledge/phases/01-auth-refactor/sub-phases/01.2-session-migration/PLAN.md

Next: /fledge:fledge-review plan 01-auth-refactor
```

## What this skill does NOT do
- Write the plan itself — that's the subagent's job
- Review the plan — `/fledge:fledge-review plan` handles that
- Read source content inline — pass references only

## Tools needed
- `Read`, `Write`, `Bash`, `Glob` (for file-tree scaffolding and verification)
- `Agent` (to spawn `fledge-planner`)
- `AskUserQuestion` (for phase id confirmation and SoT-freshness prompt)

## Related
- Subagent: `fledge-planner`
- References: `severity-rubric.md`, `source-manifest-format.md`, `context-budget.md`
- Next: `/fledge:fledge-review plan`
