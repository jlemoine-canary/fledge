# Sub-phase depth

## Default cap: 3 levels

```
Phase 1
├── Phase 1.1
│   ├── Phase 1.1.1   ← level 3, floor
│   └── Phase 1.1.2
└── Phase 1.2
```

## Rationale
- Beyond level 3 the plan becomes unreadable by a human at the final checkpoint
- Each level adds a subagent hop, which adds handoff risk
- If you genuinely need level 4, the top-level scope was wrong — split into sibling phases instead

## Escape hatch
`/fledge-plan --deep` allows level 4+. This is a conscious choice by the user; the planner must justify the depth in the PLAN.md `Scope` section.

## Numbering
- Top-level phases: `01`, `02`, `03`, ...
- Sub-phases: `01.1`, `01.2`, ...
- Sub-sub: `01.1.1`, `01.1.2`, ...
- Directory slug: `<number>-<kebab-slug>` (e.g. `01-auth-refactor`, `01.1-token-storage`)

## Decision rule: "should this be a sub-phase or a sibling?"

Make it a **sub-phase** when:
- It shares context/invariants with its parent that would be awkward to duplicate
- It can't be started until the parent's scaffolding exists

Make it a **sibling** when:
- It's independently reviewable and testable
- It could theoretically ship on its own
- Making it a child would push the tree past level 3

When in doubt, make it a sibling. Flat is better than nested for this purpose.
