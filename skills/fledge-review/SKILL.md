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

### 2. (code mode only) Compile the review package
Compile the deterministic change set per `references/review-package-format.md`: resolve base/head, run the fixed git recipe, and write `<phase-dir>/REVIEW-PACKAGE.md` (+ sibling `REVIEW-PACKAGE.patch`). This supersedes the old ad-hoc `CHANGES.md` — do not editorialize the package; reviewers form the opinions.

### 3. Round 1 — constructive
Spawn `fledge-reviewer-constructive` with:
- mode (`plan` or `code`)
- artifact path: `PLAN.md` (plan mode) or `REVIEW-PACKAGE.md` (code mode)
- snapshot path
- project `CLAUDE.md` path
- output path: `<phase-dir>/REVIEW-<MODE>-1-constructive.md`
- output template: `references/templates/review.md`

### 4. Round 2 — adversarial
Spawn `fledge-reviewer-adversarial` with same inputs + path to round-1 review.
Output: `<phase-dir>/REVIEW-<MODE>-2-adversarial.md`.

### 4b. (code mode only) Messaging review
Check the gate in `references/messaging-review.md`: the canary repo, `messaging-pr-review`'s
`SKILL.md` on disk, and a changed file that CODEOWNERS gives to `pod-messaging` or that matches
that file's send-path list. If all three hold, **read that `SKILL.md` now and follow its review
steps yourself** — no spawn, since the `Skill` tool refuses it and reviewers can't run skills —
against `BASE..HEAD` from
`REVIEW-PACKAGE.md`, and write `<phase-dir>/REVIEW-CODE-2b-messaging.md`. The reference lists the
overrides: the phase's diff, never `gh pr checkout`; no `messaging-pr-wrapup` handoff. Otherwise
print the one-line skip reason, naming any unowned files.

### 5. Round 3 — integrator
Spawn `fledge-reviewer-integrator` with rounds 1 and 2, plus `REVIEW-CODE-2b-messaging.md` or the
step-4b skip line.
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
messaging_review: <path to REVIEW-CODE-2b-messaging.md | skipped — <reason> | n/a (plan mode)>
messaging_verdict: <its risk tier + approvability verdict, for the user — never the review verdict>
```

## --sub-phases mode

If invoked with `--sub-phases`, iterate across the phase tree (leaves first). Review each sub-phase artifact in sequence — do not hold more than one sub-phase's review artifacts in your context simultaneously. Aggregate per-sub-phase verdicts into a tree-level summary returned to the caller.

Warn upfront if the tree implies >24 reviewer spawns (8+ artifacts × 3 personas):
> Reviewing <N> artifacts × 3 personas = <M> subagent runs. Estimated <X>min. Proceed?

## Context budget

- One SoT fetch per cycle (snapshot), not per persona.
- For >10-file code change sets: split the review by area (backend/frontend, or by requirement subset). The `## Area split` line in `REVIEW-PACKAGE.md` records it (see `references/review-package-format.md`).
- Reviewers receive paths, never artifact contents inline.
- Step 4b is the one round that runs in your own context. Check the budget before it, not after.

## Tools needed
- `Read`, `Write`, `Bash` (git diff, snapshot writes, history archive, the step-4b gate)
- `Agent` (3 reviewer subagents + planner/implementer re-spawn on BLOCK)
- Source-doc fetchers (matched to SoT type) for the snapshot refresh

## Related
- Subagents: `fledge-reviewer-constructive`, `fledge-reviewer-adversarial`, `fledge-reviewer-integrator`, `fledge-planner`, `fledge-implementer`
- References: `severity-rubric.md`, `sot-snapshot.md`, `templates/review.md`, `context-budget.md`, `review-package-format.md` (code mode), `messaging-review.md` (code mode, step 4b)
- Read at run time, never copied: `messaging-pr-review` (canary repo, `agent-plugins/claude-plugins/msg/`)
- Next on PASS (plan mode): `/fledge:fledge-test`
- Next on PASS (code mode): `/fledge:fledge-qa`
