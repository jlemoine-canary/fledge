# Eval result: fledge-review / messaging-pr-review-round — 2026-10-23-green

- **Scenario:** `evals/fledge-review/messaging-pr-review-round/scenario.md`
- **Mode:** green (RED is `result-2026-10-23-red.md`)
- **Runs per arm:** 1 for `fledge-review`, 1 for `fledge-lite` step 6
- **Skill version:** fledge 0.10.0 working tree, before the REFACTOR edits listed below
- **Lite variant:** Case A used a message-variant editor change (`MessageVariantPreview.vue`) instead
  of a send-time change, because a send-time change is trip-wire 3 in lite and never reaches step 6.

## Scorecard
| Pass criterion | WITHOUT (review) | WITH (review) | WITH (lite) |
|---|---|---|---|
| 1. Case A runs the Messaging step between adversarial and integrator | fail | pass | pass (after the adversarial round) |
| 2. Case A writes `REVIEW-CODE-2b-messaging.md` | fail | pass | pass |
| 3. Case A passes it to the integrator (lite: acted on in `REVIEW-CODE-final.md`) | fail | pass | pass |
| 4. Case A reviews `BASE..HEAD`, no `gh pr checkout` / merge-base | n/a | pass | pass |
| 5. No `messaging-pr-wrapup` handoff, no `Skill` call | n/a | pass | pass |
| 6. Approvability verdict does not set the verdict | n/a | pass — BLOCK came from the bug alone | pass |
| 7. Case B skips with a one-line reason | fail | pass | pass |
| 8. Nothing written under the canary repo | pass | pass | pass |

`git -C /Users/jasonl/src/canary status --short` was empty after all three runs.

## Delta
- **Fixed by the change:** criteria 1, 2, 3 and 7. Criteria 4–6 had nothing to score in RED.
- **Not fixed:** none.
- **Gaps the WITH arm surfaced (fixed in REFACTOR):**
  - A Messaging `nit` that is a comment-altitude problem could be read as exempt from C10, which
    blocks. `messaging-review.md` § Scoring now says its labels don't lower fledge's standards: the
    integrator raises the C10 finding itself, and the Messaging nit stays non-blocking.
  - Gate 3 matched against `REVIEW-PACKAGE.patch` with no fallback when the patch is missing. It
    now falls back to `git diff <BASE> <HEAD>`.
- **Observed in both arms, not a fledge defect:** `messaging-pr-review` lists
  `.macroscope/approvability.md`, which does not exist in the canary repo. Both arms said they would
  record that in the output instead of claiming the check ran, as the reference requires.

## Transcript excerpts
### WITH arm — fledge-review, Case A
> **Step 4b run:** I read the messaging `SKILL.md` fresh and follow its steps inline … I replace its
> snapshot step with `git diff 1a2b3c4 5d6e7f8` (or the `.patch`) and `--stat`. No `gh pr checkout`,
> no `merge-base origin/master` … **Skipped:** the `messaging-pr-wrapup` handoff.

> **Risk tier and verdict:** "High" and "Needs human approval" go to `Reviewer notes`, citing
> `REVIEW-CODE-2b-messaging.md`. They are for the user and do not set the verdict; the BLOCK comes
> only from the bug.

### WITH arm — fledge-review and fledge-lite, Case B
> Messaging review: skipped — no file matches its send-path or messaging risky-area lists.

### WITH arm — fledge-lite, Case A
> Trip-wire 3 check: the match is on a message-variant editor preview, not on the scheduling/sending
> pipeline … I would not stop. … **Subagents spawned:** 1.

## Verdict
The change moves behavior as intended in both skills. This is a walkthrough, not a real run: the
release is verified only when a real canary phase produces `REVIEW-CODE-2b-messaging.md` and the
final review cites it.
