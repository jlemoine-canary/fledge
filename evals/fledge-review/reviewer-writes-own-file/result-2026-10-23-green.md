# Eval result: fledge-review / reviewer-writes-own-file — 2026-10-23 (GREEN, checkout only)

- **Scenario:** `evals/fledge-review/reviewer-writes-own-file/scenario.md`
- **Mode:** mechanical, single arm. Same instrument as the RED run.
- **Version under test:** 0.10.1 (this branch), read from the checkout, not the installed cache.

## Scorecard

| Pass criterion | 0.9.1 (RED) | 0.10.1 (checkout) |
|---|---|---|
| 1. Every persona given an output path has `Write` | fail | **pass** |
| 2. Each persona writes its review with `Write` | fail: 0 of 163 | not yet run |
| 3. No persona hands the review back as text instead | fail: 21 runs | not yet run |
| 4. The file exists when the round returns | partial | not yet run; both skills now check it |

## Criterion 1

| Skill step | Output path | Written by | `Write`? |
|---|---|---|---|
| `fledge-review` step 3 | `REVIEW-<MODE>-1-constructive.md` | `fledge-reviewer-constructive` | yes |
| `fledge-review` step 4 | `REVIEW-<MODE>-2-adversarial.md` | `fledge-reviewer-adversarial` | yes |
| `fledge-review` step 4b | `REVIEW-CODE-2b-messaging.md` | the orchestrator (`## Tools needed`: `Write`) | yes |
| `fledge-review` step 5 | `REVIEW-<MODE>-final.md` | `fledge-reviewer-integrator` | yes |
| `fledge-lite` step 6 | `REVIEW-CODE-final.md` | `fledge-reviewer-adversarial`, then the orchestrator adds to it | yes |

## Why criteria 2–4 wait for a real run

Spawning `fledge:fledge-reviewer-*` from this session uses the installed cache's allowlist, which
is still 0.10.0's. A sandbox arm would score the old version. Per the fledge-writing-skills skill's
release checklist, the check is the next real run after the cache moves. The `CHANGELOG.md`
0.10.1 "Verify this release by" line names what to look for.

## REFACTOR: the loophole this opens

Once both skills check that the file exists, the cheapest way to pass the check is for the
orchestrator to save a reviewer's text-only reply itself. That is the improvised fix the RED audit
found, and it makes a file whose writer depends on which session ran. Both skills now say: re-spawn
once and do not save the text yourself.

## Verdict

GREEN on criterion 1. Not released: this is step 2 of the release checklist, unverified until a
real run's reviewer transcript shows a `Write` call to its review file.
