# Scenario: fledge-review / stale-sot-and-state-gaps

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`references/code-review-checklist.md` plan-mode items, read by `fledge-reviewer-constructive` —
whether a plan review:

- treats a source-of-truth that **predates and disagrees with** the project description as an
  `Open questions` entry (P1), instead of trusting the SoT because it is marked `sot: true`; and
- asks for the gate's **state changes and outcomes** and a **naming decision before the migration
  unit** (new item P12), instead of leaving both for code review.

Both arms run the constructive reviewer persona; the WITHOUT arm gets the 0.7.0 persona and
checklist (`<SKILL_ROOT>` = snapshot), the WITH arm the revised ones.

## Task prompt

> You are the fledge constructive reviewer, round 1, **plan mode**. Read and follow
> `<SKILL_ROOT>/agents/fledge-reviewer-constructive.md` and the references it names (paths relative
> to `<SKILL_ROOT>`; read `<SKILL_ROOT>/references/code-review-checklist.md` and
> `<SKILL_ROOT>/references/severity-rubric.md` in full).
>
> The phase directory is
> `/Users/jasonl/src/fledge/.claude/worktrees/b2b-retro-design-checks/evals/fledge-review/stale-sot-and-state-gaps/setup/`.
> It holds `PLAN.md`, the manifest `SOURCES.md`, the SoT snapshot `.sot-snapshot.md`, and
> `linear-project-P-MSG-1996.md`, a copy of manifest entry S2. **This is a sandbox:** do not call
> any MCP tool or fetch anything; those four files are the whole world. There is no project
> `CLAUDE.md`; message scheduling is risk level 5. Treat the snapshot as fresh.
>
> Write your review to `<OUT_DIR>/REVIEW-PLAN-constructive.md` using
> `<SKILL_ROOT>/references/templates/review.md`. Reply with the path only.

## Setup / context

`setup/` — a plan for a back-to-back owner gate that follows the SoT faithfully:

- **S1 (SoT, Notion), last edited 2026-08-17:** pairs only; skip pre-arrival and check-in at
  *scheduling* time; field `b2b_owner`.
- **S2 (Linear project, non-SoT), description edited 2026-09-01:** "Redesign (2026-08-31)" —
  schedule everything and filter at *send*; chains of any length; all message types.
- **`PLAN.md`:** follows S1 exactly; `## Open questions` says "None."; no enumeration of state
  changes; PR1 is the `b2b_owner` migration, with no naming decision anywhere.

`setup-subtle/` — the same files, except S2 states the send-time design plainly, without the
"Redesign (2026-08-31)" announcement. A sharper test of P1: nothing says the SoT is stale except
the dates and the disagreement.

## Pass criteria (observable)

1. A finding identifies that S1 and S2 **disagree** on the design (schedule-time vs send-time, or
   pairs vs chains, or two message types vs all).
2. That finding uses the **dates** — S1 last edited before S2's redesign — as evidence the SoT may
   be stale, rather than treating S2 as a non-SoT divergence to log and ignore.
3. That finding's fix is to **raise it under `Open questions`** (ask the user which source wins /
   whether S1 is superseded), not to silently re-plan against either source. It is
   `consequential = yes`.
4. A finding asks for the gate's **state changes with outcomes**, naming at least one of: the owner
   becomes ineligible after it has already sent; the chain splits; the chain grows after scheduling.
5. A finding asks for **new domain names** (`b2b_owner`, `b2b_dedupe`) to be settled in the plan
   before the PR1 migration, and it is `consequential = yes` (not a `nit`).

## Known failure modes (what the WITHOUT arm is expected to do)

- Reviews the plan against S1 alone, finds it traceable, and logs S2 as a non-SoT divergence that
  "does not block" (manifest rule 4).
- Flags the S2 difference but recommends re-planning to S2, i.e. silently swaps which source wins.
- Lists "cancellation" as a test-plan gap without the owner-already-sent outcome.
- Scores naming as a `nit` (the rubric's definition names "naming") or doesn't raise it at all.

## Notes

- Reproduces the back-to-back reservations project (retro 2026-10-14): the Notion design doc stopped
  at Aug 31 while the Linear description became the real source; fledge fetches the SoT fresh each
  cycle and trusts it. The owner-already-sent leak (MSG-6397) and the `b2b_owner` rename were found
  in code review after PRs were stacked on them.
- Both arms inherit `~/.claude/shared/working-agreements.md`, which the persona also tells them to
  read; since 2026-10-14 it carries *Settle state changes and names at design time* and *Design docs
  stay current or say they're superseded*. A WITHOUT arm that cites those is the "rule already
  written" comparison, and the result file must say so.
