# Checkpoint protocol

Fledge has **exactly two** user-approval checkpoints in the default pipeline:

1. **After final plan review passes** — before any code is written
2. **After QA passes** — before declaring the project done

All other stages run autonomously (subject to context-budget rules).

## Checkpoint mechanics

At each checkpoint, the orchestrator:

1. Writes a **checkpoint file** to `.fledge/checkpoints/<timestamp>-<stage>.md` summarizing:
   - What was produced (artifact paths)
   - Key decisions made
   - Any non-blocking (nit) findings deferred
   - What happens next if the user approves
2. Uses `AskUserQuestion` with **explicit options**, not freeform:
   - `approve` — continue to next stage
   - `iterate` — go another review round (user can paste feedback)
   - `abort` — stop and write a resume file

## Checkpoint file format

```markdown
# Checkpoint: <stage> — <ISO timestamp>

## Summary
<2–3 sentences>

## Artifacts
- <path> — <one-line description>

## Decisions locked in this stage
- <decision> (rationale: <why>)

## Non-blocking findings (deferred)
- <nit> — <location>

## On approval, next up
<1 sentence>
```

## Mid-pipeline failures

If any stage fails autonomously (e.g. plan review loop can't converge in 3 rounds, code review loop stalls, QA never passes), **treat it as a checkpoint** — stop, write the failure state, ask the user for direction. Do not retry indefinitely.

## Convergence caps (per stage)

- Plan review: **max 3 rounds** (constructive → adversarial → integrator). If round 3 still has consequential findings, escalate to user.
- Code review: **max 3 rounds** (same persona sequence). Same escalation rule.
- Implementation TDD loop: **max 5 iterations** per failing test. If a test won't pass in 5 tries, escalate.
- QA loop (browser and/or API, per phase surface): **max 3 rounds** of test-fail → fix → test-pass. Escalate if it still fails.
