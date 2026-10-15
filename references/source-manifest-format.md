# Source manifest format — `.fledge/SOURCES.md`

This file is the single index of all source documents for a fledge project. Every skill reads it; no skill guesses or re-derives source content.

## Rules
1. **Exactly one** source marked `sot: true`. All others `sot: false`.
2. Every entry has a **directly fetchable reference** — a URL, Linear ID, Notion page ID, Figma node URL, or local path. Skills fetch the source live; they do not rely on distilled summaries.
3. When a new source is added (`/fledge-ingest --append`), the user is asked whether it becomes the new SoT.
4. If a finding conflicts between two sources, the SoT wins. Non-SoT divergences are logged but do not block. **Exception:** when the SoT disagrees with the ticket or project description, or was last edited before the ticket's design changed, neither side wins. The plan records it under `Open questions` (checklist P1).
5. Entries with `kind: prior-work` are earlier investigations of the same problem, found by ingest's prior-work search. They are context, not requirements, and never the SoT. The planner reads them, and a plan whose design departs from one says why.

## Format

```markdown
# Source manifest

**Project:** <short name>
**Last updated:** <ISO date>

## Sources

### S1 — <human title>
- **type:** notion | linear-issue | linear-project | linear-doc | github-issue | github-pr | figma | slack-thread | file | url
- **ref:** <id or URL or path>
- **sot:** true | false
- **notes:** <one line, optional — e.g. "PRD, authoritative on email copy">
- **kind:** prior-work  <only on entries found by the prior-work search; omit otherwise>

### S2 — <human title>
- **type:** ...
- **ref:** ...
- **sot:** false
- **notes:** ...

## Prior-work search
**Searched:** <ISO date> · **Terms:** "<term>", "<term>"
- Slack: <N hits opened, M added (S<n>) | none found | not searched — <reason>>
- Notion: ...
- Linear: ...
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
| `slack-thread` | Slack read-thread tool (e.g. `slack_read_thread`) with the thread's permalink |
| `file` | `Read` |
| `url` | `WebFetch` |

Every fetch should pull **current** content, not cache. These docs change — the plan is against live state.
