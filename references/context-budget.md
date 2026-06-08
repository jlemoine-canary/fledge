# Context budget protocol

## Core principle
If the orchestrator is at 50%+ context, subagent scoping has failed. The 50% line is a **warning**; 70% is a **stop**.

## Prevention (before spawning subagents)
Every skill that spawns subagents must estimate scope upfront:

1. **Count source docs and sum their approximate size.** >50k tokens of source → split ingest into targeted fetches by section/ticket.
2. **Count plan sub-phases.** >5 sub-phases at the same level → the parent plan is doing too much; split into sibling phases instead of nesting.
3. **Scope each subagent tightly** — pass only the source-doc references and prior-artifact paths it needs, not full contents.

Prefer **references** (file paths) over **content** (file bodies) in subagent prompts. Subagents can Read what they need.

## Detection
After each subagent returns, the orchestrator checks its own context usage. Claude Code exposes context pressure via internal telemetry; a rough proxy is message count + major tool-result sizes.

## Action at thresholds

### At 50% — warn
Write a status line to the user:
> ⚠ Fledge: context at ~50%. Remaining work: `<list>`. Continuing.

Do **not** silently reduce scope. Keep going but spawn the next subagent with an even tighter brief.

### At 70% — stop and ask
1. Write current state to `.fledge/checkpoints/<timestamp>-context-stop.md` with:
   - What completed
   - What's remaining
   - The last artifact path each subagent produced
2. Ask the user:
   - (a) Continue with tighter scoping (you propose how)
   - (b) Split into a fresh session (you provide the resume prompt)
   - (c) Proceed as-is (accept risk of compaction)

**Never silently reduce scope.** Dropped work is worse than interrupted work.

## Scope-splitting heuristics
- If a plan review is too large, review one finding class at a time (correctness → security → perf → style)
- If an implementation is too large, split by test file
- If ingest is too large, ingest only the SoT first; defer secondary sources to on-demand fetch
