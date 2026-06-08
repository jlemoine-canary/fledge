# Source-of-truth snapshot protocol

To avoid re-fetching the source-of-truth multiple times per review cycle, the orchestrating skill writes one snapshot up front, and all subagents in that cycle read from the snapshot.

## Snapshot is live, not cached across sessions

The snapshot exists for the duration of a single review cycle. It is **freshly fetched** at the start of every cycle. This honors the "no distilled summaries" rule — the snapshot is the live source, captured for consistent reading by multiple subagents in the same stage.

## Snapshot path
`.fledge/phases/<phase-id>/.sot-snapshot.md`

## Snapshot format

```markdown
# SoT snapshot — <ISO timestamp>

**Source:** <S<N> entry from manifest>
**Fetched at:** <ISO timestamp>
**Fetcher tool:** <e.g. mcp__notion__notion-fetch>
**Cycle:** <N — increment each review cycle that re-fetches>

---

<Full content of the source-of-truth, as returned by the fetcher.>
```

## When to refresh the snapshot

- At the start of every new review cycle (cycle 1, cycle 2, ...) — always
- When `SOURCES.md` `Last updated` changes (someone added/changed a source mid-flow)
- When the user runs `/fledge-ingest --append` and promotes a new SoT

Reviewers do **not** refresh independently. The skill that spawns them refreshes once per cycle.

## When NOT to use the snapshot

- The planner (`fledge-planner`) fetches live for the first plan. The snapshot is a review-cycle optimization, not a planning optimization. Planning happens once; review happens N times.
- The implementer fetches live if it hits ambiguity during implementation — implementation can take long enough that the SoT may legitimately have changed.

## Stale snapshot detection

Before reading the snapshot, the reviewer checks:
1. The `Fetched at` timestamp is < 24 hours old
2. The `SOURCES.md` `Last updated` field is ≤ snapshot's `Fetched at`

If either fails, flag to the orchestrator: "snapshot stale, refresh before continuing." The orchestrator handles the refresh, not the reviewer.
