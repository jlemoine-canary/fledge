# QA — <phase-id>

**Surface:** <frontend | backend | full-stack | library-internal>
**QA type:** <browser (Playwright) | API + side-effects | CLI smoke | skipped — see rationale>

> For `library-internal` skips: state the rationale here — which test-stage coverage stands in
> for QA, and why no browser/API layer applies. See `references/qa-by-surface.md`.

## Scenarios
| # | Scenario | Surface | Requirement | Status |
|---|---|---|---|---|
| 1 | <scenario> | fe / be | <R-id> | pass / fail / blocked / skipped |
| 2 | ... | | | |

## Test files
- <path> — new
- <path> — appended to existing suite

## Environment
- Dev server: <URL>, started via `<command>`
- Test account: <how seeded>
- Test data: <fixtures or factories used>

## Notes
<Anything QA-relevant the next reviewer/implementer should know.>
