# Eval result: fledge-review / review-package-determinism — 2026-06-25-baseline

> Produced per `references/skill-eval-protocol.md`. RED→GREEN delta for the
> `references/review-package-format.md` recipe (issue #3).

- **Scenario:** `evals/fledge-review/review-package-determinism/scenario.md`
- **Mode:** delta
- **Runs per arm:** 2 (WITHOUT ×2, WITH ×2 — divergence/convergence is the measured property)

## Scorecard
| Pass criterion | WITHOUT arm | WITH arm |
|---|---|---|
| 1. Base resolved by stated deterministic rule | pass (both used merge-base) | pass |
| 2. Fixed, ordered section set | fail | pass |
| 3. `--name-status` order + status letters preserved, no annotation | fail | pass |
| 4. Second independent run is structurally identical | fail | pass |

## Delta
- **Fixed by the recipe:** criteria 2–4. The two WITH runs produced byte-identical
  `REVIEW-PACKAGE.md` (same filename, same six sections in order, same base/head, same
  `## Area split` default line) — modulo the (fixed) timestamp.
- **Not fixed:** none.
- **New rationalizations observed:** none in the WITH arm.

## Transcript excerpts
### WITHOUT arm (divergence)
- Run 1: wrote `CHANGES.md`; added editorial `## Review focus` prose and a subjective
  per-file "Notes" column; **omitted the diff** (reviewers re-run git themselves).
- Run 2: wrote `CHANGES.md` **plus a saved `.changeset.patch`**; stayed factual; no
  "Review focus" section. Both independently noted that `references/review-package-format.md`
  did not exist — one filed a background task to create it.
- → Same base, but different filename strategy, different diff handling, different section
  set. Criterion 4 fails: the two artifacts are not structurally interchangeable.

### WITH arm (convergence)
Both runs: filename `REVIEW-PACKAGE.md` (+ sibling `REVIEW-PACKAGE.patch`); base `a1b2c3d`
resolved via the documented fallback (`.base-commit` absent → `merge-base` with `origin/main`),
labeled as such; sections metadata → Commits → Files → Stats → Area split, verbatim git
output, git's file order preserved, no editorial prose. The two `REVIEW-PACKAGE.md` bodies
were identical.

## Verdict
The recipe changes behavior in the intended direction: it collapses the run-to-run structural
divergence the WITHOUT arm exhibits into a single reproducible package. Ship.
