---
name: fledge-ingest
description: Ingest source documents for a fledge project — accepts Notion pages, Linear issues/projects, GitHub issues/PRs, Figma designs, and local files/URLs. Builds or appends to .fledge/SOURCES.md and asks which source is the source-of-truth. Use when starting a new fledge project, adding a new source mid-project (--append), or when the user says "ingest sources", "add this doc", "start a project from these docs", etc.
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

### 3. Ask which is the source-of-truth

**Fresh ingest**: use `AskUserQuestion` with exactly one of the parsed sources as options. Required — there is always one SoT.

**Append mode**: for each new source, ask:
> This is a new source. Should it become the source-of-truth?
> - yes — demote current SoT, promote this one
> - no — add as non-SoT
> - cancel — don't add it

If SoT changes mid-project, warn:
> ⚠ Source-of-truth changed after planning began. Existing plans may need re-review against the new SoT. Run `/fledge:fledge-review plan` on any active plan.

### 4. Write `.fledge/SOURCES.md`

Use the format in `../../references/source-manifest-format.md`. Include a short `Last updated` ISO date.

**Fresh ingest**: scaffold the file from scratch.
**Append mode**: append new `S<N>` entries (next available id); flip the old SoT to `sot: false` if a new one was promoted; bump `Last updated`.

### 5. Summarize back to the user

```
✓ <Ingested | Appended> N sources. Source-of-truth: <title> (<ref>) [<unchanged | promoted>].
.fledge/SOURCES.md updated.
Next: /fledge:fledge-plan to start planning, or /fledge:fledge-ingest --append to add more.
```

## Scope-management

Source doc sizes can vary wildly. A single Notion page can be 40k tokens; a Figma node can blow context.

- **Do not** paste full source contents into your output. The manifest holds references; downstream skills re-fetch.
- **Do** record the approximate size (token count or character count) for each source in a comment within SOURCES.md, so downstream orchestrators can budget.

## Tools needed
- `mcp__notion__notion-fetch`, `mcp__notion__notion-search`
- `mcp__linear-server__get_issue`, `_project`, `_document`, `_list_*`
- Bash (for `gh issue view`, `gh pr view`)
- `mcp__figma__get_metadata` (schema validation without full content pull)
- `Read`, `WebFetch`
- `Write` (for SOURCES.md)
- `AskUserQuestion`

## Related
- `/fledge:fledge-plan` — next stage
- `/fledge:fledge-review plan` — re-run if SoT changed and plans exist
