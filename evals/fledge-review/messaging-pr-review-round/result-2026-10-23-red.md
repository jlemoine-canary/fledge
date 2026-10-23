# Eval result: fledge-review / messaging-pr-review-round — 2026-10-23-red

- **Scenario:** `evals/fledge-review/messaging-pr-review-round/scenario.md`
- **Mode:** red
- **Runs per arm:** 1 (the baseline skill has no conditional step, so the outcome is determined by
  the text, not by sampling)
- **Skill version:** `skills/fledge-review/SKILL.md` at fledge 0.9.1 (commit `3b2537a`)
- **What WITHOUT measured:** no rule. Nothing in 0.9.1 or the inherited working agreements names
  `messaging-pr-review`.

## Scorecard
| Pass criterion | WITHOUT arm |
|---|---|
| 1. Case A runs the Messaging step between adversarial and integrator | fail |
| 2. Case A writes `REVIEW-CODE-2b-messaging.md` | fail |
| 3. Case A passes it to the integrator | fail — integrator gets rounds 1 and 2 only |
| 4. Case A reviews `BASE..HEAD`, no `gh pr checkout` / merge-base | n/a — no Messaging step |
| 5. No `messaging-pr-wrapup` handoff, no `Skill` call | n/a — no Messaging step |
| 6. Approvability verdict does not set the verdict | n/a — no Messaging step |
| 7. Case B skips with a one-line reason | fail — nothing to skip, nothing printed |
| 8. Nothing written under the canary repo | pass |

## Transcript excerpts
### WITHOUT arm
> The walkthrough is the same for both cases. The baseline skill has no step that depends on the
> content of the diff, so nothing changes between the backend messaging diff and the frontend
> store diff.

> **Gap:** this diff touches message scheduling. `working-agreements.md` rates that "risk level 5"
> … The baseline skill has no step that acts on that. The diff gets exactly the same three generic
> reviewers as Case B, with no extra check, no Messaging- or scheduling-specific reviewer or skill,
> and no note to the reviewers.

Files it would write for Case A: `.sot-snapshot.md`, `REVIEW-CODE-1-constructive.md`,
`REVIEW-CODE-2-adversarial.md`, `REVIEW-CODE-final.md`, `review-history.md`.

## Verdict
Baseline confirmed. A send-path phase in canary gets the same review as a frontend utility fix, and
the Messaging team's review never runs. The arm noticed the risk on its own and still did nothing
about it, because the skill gave it no step to take.
