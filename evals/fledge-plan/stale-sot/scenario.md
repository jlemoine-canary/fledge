# Scenario: fledge-plan / stale-sot

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-planner` (and manifest rule 4 in `references/source-manifest-format.md`) — whether the
planner notices that the SoT predates and disagrees with the project description in the manifest,
and records that under `Open questions` instead of planning straight from the SoT.

The WITHOUT arm gets the 0.7.0 planner (`<SKILL_ROOT>` = snapshot), the WITH arm the revised one.

## Task prompt

> You are the fledge planner. Read and follow `<SKILL_ROOT>/agents/fledge-planner.md`, and write the
> plan in the shape of `<SKILL_ROOT>/references/templates/plan.md`.
>
> Phase `01-owner-gate`. The source manifest is
> `/Users/jasonl/src/fledge/.claude/worktrees/b2b-retro-design-checks/evals/fledge-review/stale-sot-and-state-gaps/setup-subtle/SOURCES.md`.
> **This is a sandbox:** do not call any MCP tool or fetch anything from the network. Fetching a
> manifest source means reading its file in that same directory: S1 (the Notion page) is
> `.sot-snapshot.md`, and S2 (the Linear project) is `linear-project-P-MSG-1996.md`. Ignore the
> `PLAN.md` in that directory; it is not yours. Do not survey a real codebase — assume a mature
> Django monorepo with `guest` and `guest_journey` apps. There is no project `CLAUDE.md`; message
> scheduling is risk level 5.
>
> Write the plan to `<OUT_DIR>/PLAN.md`. When done, reply with the path and one line listing which
> manifest sources you read.

## Setup / context

`evals/fledge-review/stale-sot-and-state-gaps/setup-subtle/`: S1 (SoT, Notion, last edited
2026-08-17) describes a schedule-time skip for pairs and two message types; S2 (Linear project,
description edited 2026-09-01) describes send-time suppression across chains of any length and
all message types. S2 states its design without announcing a redesign.

## Pass criteria (observable)

1. The arm reports reading S2, not just the SoT.
2. `PLAN.md` § Open questions has an entry naming the S1/S2 disagreement and asking the user
   which holds (or whether S1 is superseded).
3. The plan does not silently pick a side: it neither plans to S1 without mentioning S2, nor
   plans to S2 while leaving the SoT unquestioned.

## Known failure modes (what the WITHOUT arm is expected to do)

- Reads only the SoT (0.7.0 step 1 says "Read the source-of-truth live" and nothing about the
  other sources), plans a schedule-time pair gate, and writes `Open questions: None` or questions
  unrelated to the conflict.

## Notes

- Reproduces the back-to-back reservations project (retro 2026-10-14): the Notion design doc
  stopped at Aug 31 while the Linear description became the real source, and fledge fetches the
  SoT fresh each cycle and trusts it.
- Both arms inherit `~/.claude/shared/working-agreements.md`, which carries *Design docs stay
  current or say they're superseded* since 2026-10-14. Record whether the WITHOUT arm cites it.
