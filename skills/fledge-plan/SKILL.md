---
name: fledge-plan
description: Use when the user says "plan this phase", "plan <phase-id>", "break this down", or after /fledge:fledge-ingest establishes sources. Produces a staff-engineer-level PLAN.md via the fledge-planner subagent, recursing into sub-phases up to depth 3 (--deep for depth 4+).
---

# fledge-plan

Produce `PLAN.md` for a phase. Spawns `fledge-planner` subagent. If the planner identifies sub-phases, this skill recurses.

## Preconditions

- `.fledge/SOURCES.md` exists and has exactly one source-of-truth
- Project root has a `CLAUDE.md` (or the user has confirmed it has none — rare)

## Arguments

- **Phase identifier** — e.g. `01-auth-refactor`. If omitted, derive from the source-of-truth title and prompt user to confirm.
- **`--deep`** — allow sub-phase depth > 3. Planner must justify.

## Process

### 1. Resolve the phase
- If no existing `.fledge/phases/<id>/` directory, create it and this is a top-level phase at depth 1.
- If spawned recursively for a sub-phase, parent passes the sub-phase id and depth level.

### 2. Check source-of-truth freshness
The SoT may have changed since ingest. Re-fetch it and compare last-updated timestamp against `.fledge/SOURCES.md`'s `Last updated`. If changed:
- Warn the user
- Offer: "Update SOURCES.md before planning? (recommended yes)"

### 3. Spawn `fledge-planner`
Pass in a self-contained prompt with:
- Phase id, phase directory path, depth level, depth cap
- Path to `.fledge/SOURCES.md`, SoT id
- Path to project `CLAUDE.md`
- Path to parent plan (if sub-phase)
- Paths to sibling phase plans (if any, for invariant context)
- The relevant `references/` docs (severity, source-manifest, subphase-depth)

**Do NOT paste source-doc content** into the prompt. Pass the reference; the planner fetches live.

### 4. Handle the planner's output
The planner writes `PLAN.md` and returns either:
- A simple success
- A refusal (depth cap hit, ambiguity in SoT)

### 5. Recursion on sub-phases

If `PLAN.md` contains a `## Sub-phases` section:
1. For each sub-phase entry, create a sub-phase directory
2. Spawn one `fledge-planner` per sub-phase — **sequentially** unless they're genuinely independent (check dependencies in the plan)
3. If 3+ sub-phases are independent, run them in parallel (single message, multiple Agent calls) to save wall time
4. After each sub-phase returns a PLAN.md, it may itself have sub-sub-phases — recurse

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
- References: `severity-rubric.md`, `source-manifest-format.md`, `subphase-depth.md`, `context-budget.md`
- Next: `/fledge:fledge-review plan`
