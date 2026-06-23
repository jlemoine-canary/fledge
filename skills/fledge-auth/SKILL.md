---
name: fledge-auth
description: Verify that all MCP connections fledge depends on are authenticated — Notion, Linear, GitHub (via gh), Figma, Playwright. Use when starting a new fledge project, after auth errors, or whenever the user asks to "check auth" or "check connections" for fledge.
---

# fledge-auth

Verify MCP and CLI authentications required by the rest of the fledge pipeline. Fail fast with actionable setup instructions if anything is missing.

## When to run
- First step of any `/fledge:fledge` project (automatically invoked)
- Any time a fledge skill errors on an MCP call
- On demand, when the user asks

## What to check

Run each probe. Report a table with `connector → status → fix-if-broken`.

### 1. Notion
Probe: call `mcp__notion__notion-search` with a trivial query like `"fledge"`.
- Pass: any response (even empty) means auth works
- Fail: auth error → call `mcp__notion__notion-fetch` won't help; tell the user to authenticate the Notion connector in Claude Code settings

### 2. Linear
Probe: call `mcp__linear-server__list_teams`.
- Pass: teams return
- Fail: auth error → direct the user to `/mcp` to authenticate `linear-server`

### 3. GitHub
Probe: `gh auth status` via Bash.
- Pass: shows logged-in user
- Fail: `gh auth login` — tell the user to run it

### 4. Figma
Probe: Figma MCP server is in the plugin system under `plugin:figma`. Check tool availability only — do not fetch a real design.
- If the `mcp__figma__*` tools aren't loadable via ToolSearch, instruct: install the `figma` plugin (`/plugin` → claude-plugins-official → figma).

### 5. Playwright (only needed for frontend / full-stack QA)
Playwright drives **browser QA** and is required only when a phase's surface is `frontend` or
`full-stack` (see `references/qa-by-surface.md`). Backend-only and library-internal phases QA
without it. So treat this as a **conditional, non-blocking** check.

Probe: `mcp__playwright__browser_close` (a no-op when no browser open) is safe to call; or simply verify the tool schemas load.
- If available → ✓.
- If not available → report `not installed` with the fix (install Playwright MCP: `claude mcp add playwright`, then restart), but **do not block the pipeline on it**. Frontend/full-stack QA will need it; backend/library QA will not. Note it as a deferred requirement rather than a hard failure.

## Output

Report to the user:

```markdown
## fledge auth check

| Connector | Status | Action |
|---|---|---|
| Notion | ✓ | — |
| Linear | ✗ | Run /mcp and authenticate linear-server |
| GitHub (gh) | ✓ | — |
| Figma MCP | ✓ | — |
| Playwright MCP | ⚠ not installed | Needed only for frontend/full-stack QA: `claude mcp add playwright`, then restart |

Next: <either "all good, proceed to /fledge:fledge-ingest" or "fix the above first">
```

If a **required** connector fails (Notion, Linear, GitHub, Figma — the ones the source docs and review stages depend on), **STOP the pipeline**. Do not continue to ingest / plan / etc.

Playwright is the exception: it's **only** required for `frontend`/`full-stack` QA. If it's the
only thing missing, do **not** block — report it as a deferred requirement (⚠) and let the
pipeline proceed. `/fledge:fledge-qa` re-checks it when it actually classifies a UI phase.

## Tools needed
- Bash (for `gh auth status`)
- ToolSearch (to probe tool availability)
- The MCP tools themselves for active probes

## Notes

Auth state can change between sessions. This check is cheap; run it whenever there's doubt. Do not cache results.
