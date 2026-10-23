# Eval result: fledge-review / messaging-pr-review-round — 2026-10-23-green-codeowners

- **Scenario:** `evals/fledge-review/messaging-pr-review-round/scenario.md` (Cases A–D)
- **Mode:** green, after gate 3 changed from "messaging-pr-review's lists" to "CODEOWNERS gives
  it to `pod-messaging`, or its send-path list matches"
- **Runs per arm:** 1 for `fledge-review` (A–D), 1 for `fledge-lite` step 6 (A with
  `frontend/hotels/src/chat/ChatConversations.vue`, B, C; D is trip-wire 3 in lite)
- **How owners were resolved:** both arms ran the reference's recipe verbatim against the real
  canary `CODEOWNERS`, feeding the case's file list in place of `git diff --name-only` (the SHAs
  are not real commits).

## Scorecard
| Pass criterion | review | lite |
|---|---|---|
| 1. A runs the step between adversarial and integrator (lite: after the round) | pass | pass |
| 2. A writes `REVIEW-CODE-2b-messaging.md` | pass | pass |
| 3. A's file reaches the integrator (lite: `REVIEW-CODE-final.md`) | pass | pass |
| 4. `BASE..HEAD`, no `gh pr checkout` / merge-base | pass | pass |
| 5. No `messaging-pr-wrapup` handoff, no `Skill` call | pass | pass |
| 6. Approvability verdict does not set the verdict | pass | pass |
| 7. B skips, naming both `DateNowStore` files as unowned | pass | pass |
| 8. Nothing written under the canary repo | pass | pass |
| 9. C runs on CODEOWNERS: line 160 beats line 158 | pass | pass |
| 10. D runs on the send-path list despite Arrivals-Departures ownership | pass | n/a |

`git -C /Users/jasonl/src/canary status --short` was empty after both runs.

## Delta
- **Fixed by the gate change:** C would have been skipped under the list-only gate (an AI chat
  admin view is not on the send path); it now runs. A Messaging-owned inbox change in lite now
  runs for the same reason.
- **Kept:** D still runs, through the send-path test, though CODEOWNERS gives it to another team.
- **Gap fixed in REFACTOR:** the skip-line template ended in a period, and the lite arm put
  `(unowned: …)` after it. The template no longer ends in a period and says to append the suffix.
- **Found while building the recipe, before either run:** `git check-ignore` with CODEOWNERS
  patterns reports the first ignored parent directory, so nested rules lose. On canary it gives
  `internal_support/views/ai/ai_chat_agent_models_view.py` to Internal Tools (line 158) instead of
  Messaging (line 160). The reference ships a matcher that applies the last matching rule, and warns against
  `check-ignore`.
- **Pre-existing, outside this change:** the constructive and adversarial reviewer personas have
  no `Write` tool, yet both skills hand them an output path.
- **In `messaging-pr-review`, not fledge's to fix:** it lists `.macroscope/approvability.md`,
  which does not exist in canary; both arms recorded that in the output as the reference requires.

## Transcript excerpts
### review, Case B
> `Messaging review: skipped — no Messaging-owned or send-path files (unowned: frontend/packages/shared/utilities/DateNowStore.ts, frontend/packages/shared/utilities/DateNowStore.test.ts).`

### review, Case C
> Line 160, `/backend/canary/internal_support/views/ai/ai_chat_*.py`, is the last match and gives
> this file to pod-messaging. This is the nested-rule case the reference warns `check-ignore` gets
> wrong.

### review, Case D
> CODEOWNERS line 300, `/backend/canary/guest/`, gives the file to
> `@canary-technologies-corp/pod-arrivals-departures`. It is not Messaging-owned. … Send-path:
> `reservation_disable_scheduled_messages.py` is a scheduled-message code path.

## Verdict
The gate behaves as intended on all four cases in both skills. Still a walkthrough: the release is
verified by a real canary phase that produces `REVIEW-CODE-2b-messaging.md`.
