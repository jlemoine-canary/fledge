---
name: fledge-review
description: Use when the user says "review the plan/code for <phase>", or after /fledge:fledge-plan or /fledge:fledge-implement completes. Three-round review (constructive → adversarial → integrator) iterating up to 3 cycles until PASS; pass mode "plan" to review a PLAN.md or "code" to review an implementation.
---

# fledge-review

Three-persona review for either a plan or an implementation. Replaces the earlier `/fledge:fledge-review-plan` and `/fledge:fledge-review-code` skills.

## Arguments

- **mode** (required): `plan` or `code`
- **phase id** (optional): defaults to most recent phase with the relevant artifact (`PLAN.md` for plan mode, `IMPLEMENTATION.md` for code mode)
- **`--sub-phases`**: also review every sub-phase artifact below this one

## Preconditions

- `plan` mode: phase dir has `PLAN.md`
- `code` mode: phase dir has `IMPLEMENTATION.md` and tests are passing
- `.fledge/SOURCES.md` exists with exactly one source-of-truth

This skill returns a verdict — it does NOT checkpoint. The orchestrator (`/fledge:fledge`) decides when to checkpoint based on the verdict.

## Process

### 1. Refresh SoT snapshot for this cycle
Fetch the source-of-truth live using the fetcher matched to its type. Write to `.fledge/phases/<id>/.sot-snapshot.md` per `references/sot-snapshot.md` (timestamp, source ref, fetcher, cycle number). All three reviewers in this cycle read from the snapshot — no independent re-fetches.

### 2. (code mode only) Compile the change set
Run `git diff` against the phase's starting commit. Write the file list and stats to `<phase-dir>/CHANGES.md` for reviewers.

### 3. Round 1 — constructive
Spawn `fledge-reviewer-constructive` with:
- mode (`plan` or `code`)
- artifact path: `PLAN.md` (plan mode) or `CHANGES.md` (code mode)
- snapshot path
- project `CLAUDE.md` path
- output path: `<phase-dir>/REVIEW-<MODE>-1-constructive.md`
- output template: `references/templates/review.md`

### 4. Round 2 — adversarial
Spawn `fledge-reviewer-adversarial` with same inputs + path to round-1 review.
Output: `<phase-dir>/REVIEW-<MODE>-2-adversarial.md`.

### 5. Round 3 — integrator
Spawn `fledge-reviewer-integrator` with rounds 1 and 2.
Output: `<phase-dir>/REVIEW-<MODE>-final.md` with verdict `PASS`, `BLOCK`, or `ESCALATE`.

### 6. Handle the verdict

- **PASS** — append to `<phase-dir>/review-history.md`, return verdict to caller.
- **BLOCK** — re-spawn the appropriate author subagent:
  - plan mode → `fledge-planner` with the final review as required fixes; planner writes revised PLAN.md
  - code mode → `fledge-implementer` with the final review as required fixes; implementer updates code + IMPLEMENTATION.md
  Then archive cycle reviews into `<phase-dir>/review-history/cycle-N/` and loop to step 1 for cycle N+1.
- **ESCALATE** — return ESCALATE to caller. The orchestrator surfaces it.

### 7. Iteration cap

Maximum 3 review cycles. On cycle 3 BLOCK, treat as ESCALATE.

The integrator persona has built-in anti-loop logic — if it detects this is the third integration for the same artifact, it issues ESCALATE rather than another BLOCK.

## Return value

```yaml
verdict: PASS | BLOCK | ESCALATE
cycles_used: <N>
artifact_path: <path to PLAN.md or IMPLEMENTATION.md>
final_review_path: <path to REVIEW-<MODE>-final.md>
consequential_findings_remaining: <N — 0 on PASS>
non_blocking_nits: <N>
```

## --sub-phases mode

If invoked with `--sub-phases`, iterate across the phase tree (leaves first). Review each sub-phase artifact in sequence — do not hold more than one sub-phase's review artifacts in your context simultaneously. Aggregate per-sub-phase verdicts into a tree-level summary returned to the caller.

Warn upfront if the tree implies >24 reviewer spawns (8+ artifacts × 3 personas):
> Reviewing <N> artifacts × 3 personas = <M> subagent runs. Estimated <X>min. Proceed?

## Context budget

- One SoT fetch per cycle (snapshot), not per persona.
- For >10-file code change sets: split the review by area (backend/frontend, or by requirement subset). Document the split in `CHANGES.md`.
- Reviewers receive paths, never artifact contents inline.

## Tools needed
- `Read`, `Write`, `Bash` (git diff, snapshot writes, history archive)
- `Agent` (3 reviewer subagents + planner/implementer re-spawn on BLOCK)
- Source-doc fetchers (matched to SoT type) for the snapshot refresh

## Related
- Subagents: `fledge-reviewer-constructive`, `fledge-reviewer-adversarial`, `fledge-reviewer-integrator`, `fledge-planner`, `fledge-implementer`
- References: `severity-rubric.md`, `sot-snapshot.md`, `templates/review.md`, `context-budget.md`
- Next on PASS (plan mode): `/fledge:fledge-test`
- Next on PASS (code mode): `/fledge:fledge-qa`
