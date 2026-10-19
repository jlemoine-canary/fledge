# Eval result: fledge-lite / small-ticket-overhead — 2026-10-19-red

- **Scenario:** `evals/fledge-lite/small-ticket-overhead/scenario.md`
- **Mode:** red (WITHOUT arm only — the baseline this skill exists to fix)
- **Runs per arm:** 1 (the result is not borderline: 9 spawns against a ceiling of 2)

**What this measures.** The WITHOUT arm is the fledge plugin at 0.8.0, which has no small path.
This is "fledge as it is" vs. (later) "fledge plus a lite path" — not "no skill" vs. "skill". The
baseline is a working pipeline that is the wrong size for the ticket, and it says so itself.

## Scorecard

| Pass criterion | WITHOUT arm |
|---|---|
| 1. Spawn count ≤ 2 | **fail** — 9 on the expected path, ≈35 at the loop caps |
| 2. No planner subagent | **fail** — `fledge-planner` at stage 5 |
| 3. No implementer subagent | **fail** — `fledge-implementer` at stage 9 |
| 4. One review round, adversarial only | **fail** — constructive → adversarial → integrator, on the plan *and* again on the code |
| 5. Exactly one user checkpoint | **fail** — 2 (plan approval, ship) |
| 6. No source manifest stage | **fail** — runs `/fledge:fledge-ingest`, writes `.fledge/SOURCES.md` for one source |
| 7. Pre-flight survives | **pass** — worktree off `origin/<default>`, `jlemoine/MSG-6639/<slug>`, GPG verified |
| 8. Mutation check present and mandatory | **fail** — no mutation step anywhere in the 12 stages |
| 9. Escalation trip-wires named before planning | **fail** — 13 stop conditions, none of them a size/risk gate; vacuous, there is nothing to escalate *to* |
| 10. Closing report of kept vs. skipped | **fail** — no such artifact |

**1 / 10.**

## Delta

- **Fixed by the skill:** n/a — RED only. GREEN is `result-2026-10-19-green.md`.
- **Baseline's own verdict, unprompted:** the arm ended with a proportionality note — that the
  orchestrator spends 9 subagents and 2 checkpoints on a change the Sentry trace already
  localizes to three files, and that *"the skills expose no lite path"*. The failure is not that
  the agent misjudged the ticket; it judged it correctly and had nowhere to put that judgment.
  That is the strongest possible RED: the gap is in the library, not in the reasoning.
- **What the baseline got right, and lite must not lose:** it named the prior-work search and the
  comment pass as the two stages it would least want dropped from a trim, *because both are retro
  action items that exist precisely because they got skipped on work that looked small*. Criteria
  7–10 exist for this reason. Lite keeps the comment pass (via the reviewer's C10) and keeps the
  pre-flight; it drops the prior-work search, and that drop is named in the closing report rather
  than left silent.

## Transcript excerpts

### WITHOUT arm — the spawn table
> | 1 | `fledge-planner` | 5 |
> | 2–4 | constructive / adversarial / integrator (plan) | 6 |
> | 5 | `fledge-implementer` | 9 |
> | 6–8 | constructive / adversarial / integrator (code) | 10 |
> | 9 | `fledge-qa-engineer` | 11 |
> **Total: 9.**

### WITHOUT arm — checkpoints
> 1. **Checkpoint 1**, after the plan review returns PASS (step 7).
> 2. **Checkpoint 2**, after QA returns PASS (step 12).
> Both are mandatory: the orchestrator's design principles say "Do NOT skip checkpoints."

## Verdict

Baseline fails 9 of 10 criteria and diagnoses its own failure. Proceed to GREEN.
