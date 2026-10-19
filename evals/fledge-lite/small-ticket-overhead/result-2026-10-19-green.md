# Eval result: fledge-lite / small-ticket-overhead — 2026-10-19-green

- **Scenario:** `evals/fledge-lite/small-ticket-overhead/scenario.md`
- **Mode:** delta (RED captured separately in `result-2026-10-19-red.md`)
- **Runs per arm:** 1 each. Not borderline — 9 spawns vs. 1, 2 checkpoints vs. 1, 6 review rounds
  vs. 1. A second run was not needed to separate the arms.

**What this measures.** WITHOUT = fledge 0.8.0 (no small path). WITH = the same tree plus
`skills/fledge-lite/`. Not "no skill" vs. "skill": the baseline is a working pipeline that is the
wrong size for the ticket.

## Scorecard

| Pass criterion | WITHOUT arm | WITH arm |
|---|---|---|
| 1. Spawn count ≤ 2 | fail — 9 (≈35 at caps) | **pass — 1** |
| 2. No planner subagent | fail | **pass** |
| 3. No implementer subagent | fail | **pass** |
| 4. One review round, adversarial only | fail — 6 rounds (3 plan + 3 code) | **pass — 1** |
| 5. Exactly one user checkpoint | fail — 2 | **pass — 1, the ship checkpoint** |
| 6. No source manifest stage | fail — writes `SOURCES.md` | **pass** |
| 7. Pre-flight survives | pass | **pass** |
| 8. Mutation check present and mandatory | fail — absent | **pass — 3 mutations, backup-not-checkout, blocking** |
| 9. Trip-wires named before planning | fail — vacuous | **pass — swept at routing and again at step 3** |
| 10. Closing report of kept vs. skipped | fail — absent | **pass — full table, prior-work search named** |

**1 / 10 → 10 / 10.**

## Delta

- **Fixed by the skill:** criteria 1–6 (the trim) and 8–10 (the guard). Nine of ten.
- **Not fixed:** none.
- **New rationalizations observed:** none. The WITH arm reproduced the rationalization list and
  applied it pre-emptively — it labelled trip-wire 4 "the likeliest" on this ticket and said it
  would confirm the ticket's own "no API change" claim against the code before trusting it, which
  is the skill's instruction turned into a specific suspicion about a specific ticket.

### Behavioral deltas no criterion scores

- **The guard criteria did not come for free.** Criteria 7–10 exist because a lite path that drops
  the pre-flight or the mutation check is worse than the full pipeline, not cheaper. The WITH arm
  kept all four *and* argued for them: it justified the prior-work-search skip on the ground that
  the failing site is pinned by a stack trace, then set a condition that would reverse the call
  ("if the fix is really about who owns channel availability, that is design territory, I do the
  search anyway"). A named, reversible skip is the outcome the step-10 report was designed to
  produce.
- **It briefed around the persona mismatch unprompted** — stated "you are the only round" and gave
  the reason (the persona is written as round 2 of 3 and would otherwise hunt for a
  `REVIEW-CODE-1-constructive.md` that does not exist).
- **It recommended `qa` before `pr`** at the checkpoint, reasoning that the symptom ("the only way
  out is a page reload") is browser-level and unit tests assert it only by proxy. Lite does not
  run QA; offering it with a reason is the intended behavior.

## Gap found by this run, and closed

The WITH arm added a `## Surface` field to `PLAN.md` that the skill's four-section plan shape did
not list, because the test kind at step 4 and any later QA or escalation routing depend on it.
That was a real omission. `## Surface` is now in the step-3 plan shape, with the note that a second
value there is trip-wire 4. Re-running is not required to credit criteria 1–10 — the field changes
none of them — but the fixture now covers a skill that includes it.

## Transcript excerpts

### WITHOUT arm
> **Total: 9.** … 1. **Checkpoint 1**, after the plan review returns PASS. 2. **Checkpoint 2**,
> after QA returns PASS. … the skills expose no lite path.

### WITH arm
> **Run `/fledge:fledge-lite MSG-6639`, not `/fledge:fledge`.**
> … **Counts: 10 stages · 1 subagent (`fledge-reviewer-adversarial`) · 1 planned user stop ·
> 1 review round · 10 trip-wires plus 6 named non-trip-wire stops.**

## Verdict

The skill changes behavior in the intended direction on every criterion, and the guard criteria
hold. Ship. Pair with `escalation-trip-wire`, which measures the refusal this fixture cannot.
