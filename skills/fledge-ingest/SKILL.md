---
name: fledge-ingest
description: Use when starting a new fledge project, adding a source mid-project (--append), or when the user says "ingest sources", "add this doc", or "start a project from these docs". Accepts Notion pages, Linear issues/projects, GitHub issues/PRs, Figma designs, and local files/URLs; records which one is the source-of-truth in .fledge/SOURCES.md.
---

# fledge-ingest

Accept a list of source documents from the user, verify each is fetchable, and write or append to `.fledge/SOURCES.md`. Ask which is the source-of-truth.

## Modes

- **Fresh ingest** (default): no `.fledge/SOURCES.md` exists, or user wants to start over (`--overwrite`)
- **Append** (`--append`): an existing manifest is present; the user is adding one or more sources to it. Asks per new source whether it should become the new SoT.

## Preconditions

- `/fledge:fledge-auth` has passed (or equivalent check done)
- The user has supplied at least one source reference

## Process

### 1. Parse inputs
The user provides source references in any format — inline URLs, Linear IDs, file paths, a freeform list. Parse them into typed entries:

| Input pattern | Type |
|---|---|
| `notion.so/...` or a Notion page ID | `notion` |
| `<TEAM>-<NUMBER>` (e.g. `STAY-2122`) | `linear-issue` |
| Linear project URL | `linear-project` |
| `github.com/.../issues/N` | `github-issue` |
| `github.com/.../pull/N` | `github-pr` |
| `figma.com/design/...` or `figma.com/make/...` | `figma` |
| Local path | `file` |
| Other URL | `url` |

Unclear inputs → ask the user for clarification (but batch all clarifications into one question, not one per source).

### 2. Verify each source fetches
Run the matching fetch tool for each entry (see `../../references/source-manifest-format.md`). Confirm it returns real content. Do NOT cache the content — we re-fetch on every plan/review.

If a fetch fails:
- Auth error → refer user to `/fledge:fledge-auth`
- Not found / permission denied → ask user to double-check the reference
- Rate limit → back off and retry once

### 3. Search for prior design work (fresh ingest)
The supplied sources are what the user thought to hand over, not everything that exists. Before the manifest is final, search for earlier investigations of the same problem: design notes, abandoned or canceled tickets, triage threads. Back-to-back reservations spent its first week on a narrower design than the one a teammate had written up a month earlier, and no supplied source linked to it.

- Derive 2–4 search terms from the supplied sources' problem statement: the domain noun and the symptom ("consecutive stays", "duplicate message"), not only the project's own name or abbreviation.
- Search **all three** surfaces: Slack, Notion (`notion-search`) and Linear (issue search, including canceled and done issues). A surface you can't reach is recorded as not searched, with the reason. Never skip one silently.
- Open each plausible hit. Add a hit that discusses the same problem (a design, an attempt, a reason something failed) as a non-SoT entry with `kind: prior-work`, and say in its `notes` what it proposes or why it failed. Skip hits that only share a keyword.
- Never make prior work the SoT on your own. If it contradicts the SoT, say so in its `notes`; the planner deals with it.
- Record the terms and per-surface results in the manifest's `## Prior-work search` section, including when nothing turned up.

**Append mode** skips this step unless the user asks for it.

### 4. Ask which is the source-of-truth

**Fresh ingest**: use `AskUserQuestion` with the parsed sources as options (the ones the user supplied, never prior work); the user picks exactly one. Required — there is always one SoT.

**Append mode**: for each new source, ask:
> This is a new source. Should it become the source-of-truth?
> - yes — demote current SoT, promote this one
> - no — add as non-SoT
> - cancel — don't add it

If SoT changes mid-project, warn:
> ⚠ Source-of-truth changed after planning began. Existing plans may need re-review against the new SoT. Run `/fledge:fledge-review plan` on any active plan.

### 5. Write `.fledge/SOURCES.md`

Use the format in `../../references/source-manifest-format.md`. Include a short `Last updated` ISO date.

**Fresh ingest**: scaffold the file from scratch.
**Append mode**: append new `S<N>` entries (next available id); flip the old SoT to `sot: false` if a new one was promoted; bump `Last updated`.

### 6. Summarize back to the user

```
✓ <Ingested | Appended> N sources. Source-of-truth: <title> (<ref>) [<unchanged | promoted>].
Prior work: <N added (S3, S4) | none found> — searched <Slack, Notion, Linear> for "<terms>".
.fledge/SOURCES.md updated.
Next: /fledge:fledge-plan to start planning, or /fledge:fledge-ingest --append to add more.
```

## Scope-management

Source doc sizes can vary wildly. A single Notion page can be 40k tokens; a Figma node can blow context.

- **Do not** paste full source contents into your output. The manifest holds references; downstream skills re-fetch.
- **Do** record the approximate size (token count or character count) for each source in a comment within SOURCES.md, so downstream orchestrators can budget.

## Tools needed
- `mcp__notion__notion-fetch`, `mcp__notion__notion-search`
- A Slack search tool (e.g. `slack_search_public_and_private`), for the prior-work search
- `mcp__linear-server__get_issue`, `_project`, `_document`, `_list_*`
- Bash (for `gh issue view`, `gh pr view`)
- `mcp__figma__get_metadata` (schema validation without full content pull)
- `Read`, `WebFetch`
- `Write` (for SOURCES.md)
- `AskUserQuestion`

## Related
- `/fledge:fledge-plan` — next stage
- `/fledge:fledge-review plan` — re-run if SoT changed and plans exist
