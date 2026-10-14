# Scenario: fledge-ingest / prior-design-search

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-ingest` (and `references/source-manifest-format.md`) — whether ingest looks for earlier
investigations of the same problem before the manifest is final, and records what it finds as
non-SoT sources the planner will read.

Both arms run *a* version of `fledge-ingest`: the WITHOUT arm gets the version before the change
(`<SKILL_ROOT>` = a 0.7.0 snapshot), the WITH arm gets the revised one. This measures "ingest as it
was" vs "ingest with the prior-design search", not "no skill" vs "skill".

## Task prompt

> You are running the fledge ingest stage for a new project. Read and follow
> `<SKILL_ROOT>/skills/fledge-ingest/SKILL.md` and the references it points to (paths are relative
> to `<SKILL_ROOT>`).
>
> The user supplied two sources and has already answered the source-of-truth question:
> - Linear project **P-MSG-1996** — "Back-to-back reservation handling". **This is the SoT.**
> - Notion page `b2b-reservation-design` — the design doc.
>
> **This is a sandbox. Do not call any MCP tool or reach any network service.** The connected
> services are simulated by files under
> `evals/fledge-ingest/prior-design-search/workspace/` (in the fledge repo at
> `/Users/jasonl/src/fledge/.claude/worktrees/b2b-retro-design-checks`):
> - Fetching a supplied source = `Read` its file under `workspace/supplied/`.
> - Slack, Notion and Linear are searchable: searching one of them = `Grep`/`Glob` over
>   `workspace/slack/`, `workspace/notion/` or `workspace/linear/` respectively, and opening a
>   result = `Read`.
> - Do not ask the user anything; every answer you would need is above.
>
> Write the manifest to `<OUT_DIR>/SOURCES.md`. When done, report every search and fetch you
> performed (tool + pattern/path) and paste the final manifest.

## Setup / context

- `workspace/supplied/` — the two user-supplied sources. The SoT describes a narrow MVP
  ("pairs; skip scheduling for the second reservation").
- `workspace/slack/2026-07-21-msg-eng-guest-journey-dedupe-notes.md` — a teammate's design notes
  from a month before the project: schedule everything, filter at send, chains longer than two.
  **The prior design.**
- `workspace/linear/MSG-4410.md` — a canceled March ticket that tried the schedule-time skip and
  broke when a chain grew. **The abandoned attempt.**
- Everything else under `workspace/` is noise (an unrelated PMS thread, lunch, a CSS ticket, an
  onboarding FAQ, a message-type reference page).

## Pass criteria (observable)

1. The arm's report shows at least one search over `workspace/slack/`, one over
   `workspace/notion/` and one over `workspace/linear/` **before** the manifest was written.
2. `SOURCES.md` lists the Jul 21 Slack thread as a source with `sot: false`.
3. `SOURCES.md` lists MSG-4410 as a source with `sot: false`.
4. Those entries say in the manifest that they are prior investigations / earlier design work —
   context for the planner, not requirements.
5. The Linear project is still the only `sot: true` entry, and none of the noise files was added.
6. The manifest records which surfaces were searched and with what terms, so "nothing found" and
   "didn't search" are distinguishable.

## Known failure modes (what the WITHOUT arm is expected to do)

- Fetches the two supplied sources, writes a two-entry manifest, and never searches.
- Searches only when a supplied source links to something (neither links to the Slack thread or
  MSG-4410).
- Finds the prior work but adds it as an untyped entry indistinguishable from the supplied design
  doc, or promotes it to SoT.

## Notes

- Reproduces the back-to-back reservations project (retro 2026-10-14): a teammate's Jul 21 design
  notes held the final "schedule everything, filter at send" design, and the project spent its
  first week on the narrower pairs MVP the supplied sources describe. People and ticket numbers in
  the workspace are fictional except the project id.
- Both arms inherit the global `CLAUDE.md`, which points at `~/.claude/shared/working-agreements.md`;
  since 2026-10-14 that file carries a *Prior design work first* bullet. A WITHOUT arm that reads it
  may search anyway — record whether it did.
- Hermetic by construction: the prompt forbids MCP calls. Discard a run that reports one.
- The sandbox lets an arm list every workspace file, which a real Slack/Notion/Linear search can't
  do. Score criterion 1 on the term searches the arm reports, and check that each added entry came
  from one of them, not from the listing.
