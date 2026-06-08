# Source manifest format — `.fledge/SOURCES.md`

This file is the single index of all source documents for a fledge project. Every skill reads it; no skill guesses or re-derives source content.

## Rules
1. **Exactly one** source marked `sot: true`. All others `sot: false`.
2. Every entry has a **directly fetchable reference** — a URL, Linear ID, Notion page ID, Figma node URL, or local path. Skills fetch the source live; they do not rely on distilled summaries.
3. When a new source is added (`/fledge-ingest --append`), the user is asked whether it becomes the new SoT.
4. If a finding conflicts between two sources, the SoT wins. Non-SoT divergences are logged but do not block.

## Format

```markdown
# Source manifest

**Project:** <short name>
**Last updated:** <ISO date>

## Sources

### S1 — <human title>
- **type:** notion | linear-issue | linear-project | linear-doc | github-issue | github-pr | figma | file | url
- **ref:** <id or URL or path>
- **sot:** true | false
- **notes:** <one line, optional — e.g. "PRD, authoritative on email copy">

### S2 — <human title>
- **type:** ...
- **ref:** ...
- **sot:** false
- **notes:** ...
```

## Fetching guidance (by type)

| type | Tool |
|---|---|
| `notion` | `mcp__notion__notion-fetch` with the page ref |
| `linear-issue` | `mcp__linear-server__get_issue` |
| `linear-project` | `mcp__linear-server__get_project` |
| `linear-doc` | `mcp__linear-server__get_document` |
| `github-issue` / `github-pr` | `gh` CLI via Bash |
| `figma` | `mcp__figma__get_design_context` (Code Connect plugin) |
| `file` | `Read` |
| `url` | `WebFetch` |

Every fetch should pull **current** content, not cache. These docs change — the plan is against live state.
