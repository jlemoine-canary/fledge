# Scenario: fledge-review / messaging-pr-review-round

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-review` (code mode) and `fledge-lite` (step 6) — whether a phase in the canary repo that
changes Messaging code also gets the Messaging team's own review, `messaging-pr-review`, and whether
a phase outside Messaging code skips it with a stated reason.

`messaging-pr-review` lives in the canary repo at
`agent-plugins/claude-plugins/msg/skills/messaging-pr-review/SKILL.md` and is installed under
`~/.claude/plugins/cache/canary-plugins/msg/<version>/`. It sets `disable-model-invocation: true`,
so it never appears in an agent's skill list and the `Skill` tool cannot run it. An agent that has
not been told it exists will not find it.

This is a **walkthrough** fixture: the arm reads the skill and states what it would do, step by
step, without spawning reviewers or writing files. It scores the orchestrator's plan for the
review, which is where this step lives. It does not prove the step's findings are any good; a real
run does that.

## Task prompt

> You are the fledge orchestrator. Walk through `/fledge:fledge-review code <phase>` for each case
> below, using the skill file at `<SKILL PATH>`. Do not spawn subagents, run git, or write files —
> this is a dry run. You may read any file the skill tells you to read.
>
> For each case, list in order: every step you take; every file you would write under the phase
> dir, with its name; every input you pass to the integrator; and any skill or step you skip, with
> the reason you would print.

## Setup / context

Both cases: the project is a worktree of `git@github.com:canary-technologies-corp/canary.git`, at
`/Users/jasonl/src/canary` (read-only for this run). `.fledge/SOURCES.md` exists with one
source-of-truth. Tests pass. The phase dir has `IMPLEMENTATION.md`, and step 2 has written this
`REVIEW-PACKAGE.md`:

**Case A — phase `MSG-7001`:**

```markdown
- **Base:** `1a2b3c4` (recorded .base-commit)
- **Head:** `5d6e7f8`
- **Diff range:** `1a2b3c4..5d6e7f8`
- **Full diff:** `REVIEW-PACKAGE.patch`
## Files
M	backend/canary/guest_journey/events/event_specs/guest_journey_message/guest_journey_message_rescheduled.py
M	backend/canary/guest_journey/tests/test_guest_journey_message_rescheduled.py
```

The patch changes how a rescheduled `GuestJourneyMessage` computes its new send time.

**Case B — phase `MSG-7002`:**

```markdown
- **Base:** `9a8b7c6` (recorded .base-commit)
- **Head:** `5f4e3d2`
- **Diff range:** `9a8b7c6..5f4e3d2`
- **Full diff:** `REVIEW-PACKAGE.patch`
## Files
M	frontend/packages/shared/utilities/DateNowStore.ts
A	frontend/packages/shared/utilities/DateNowStore.test.ts
```

The patch fixes a stale-timestamp bug in a shared frontend store. No prompts, models, API
contracts, auth, flags or config are touched.

**Case C — phase `MSG-7003`:**

```markdown
- **Base:** `2b3c4d5` (recorded .base-commit)
- **Head:** `6e7f8a9`
- **Diff range:** `2b3c4d5..6e7f8a9`
- **Full diff:** `REVIEW-PACKAGE.patch`
## Files
M	backend/canary/internal_support/views/ai/ai_chat_agent_models_view.py
```

The patch fixes pagination in an internal support view that lists AI chat agent models. It does
not touch sending.

**Case D — phase `MSG-7004`:**

```markdown
- **Base:** `3c4d5e6` (recorded .base-commit)
- **Head:** `7f8a9b0`
- **Diff range:** `3c4d5e6..7f8a9b0`
- **Full diff:** `REVIEW-PACKAGE.patch`
## Files
M	backend/canary/guest/events/reservation_disable_scheduled_messages.py
```

The patch changes which of a reservation's scheduled messages get disabled when the event fires.

## Pass criteria (observable)

1. **Case A runs the Messaging step**, after the adversarial round and before the integrator, by
   reading `messaging-pr-review`'s `SKILL.md` (repo path first) and following its review steps.
2. **Case A writes `<phase-dir>/REVIEW-CODE-2b-messaging.md`.**
3. **Case A passes that file to the integrator** as one more review to reconcile.
4. **Case A reviews `BASE..HEAD` from `REVIEW-PACKAGE.md`** — no `gh pr checkout`, no merge-base
   with `origin/master`.
5. **Case A does not hand off to `messaging-pr-wrapup`** or apply fixes as part of the step, and
   does not try to run `messaging-pr-review` through the `Skill` tool.
6. **The approvability verdict does not set PASS / BLOCK / ESCALATE.** A bug is consequential, a
   suggestion is scored by the rubric, a question goes to Reviewer notes, a nit stays non-blocking.
7. **Case B skips the step with a one-line reason** naming the gate that failed, and names
   `DateNowStore.ts` and its test as unowned (canary's CODEOWNERS has no rule for them).
8. **Nothing under the canary repo is written**, and `messaging-pr-review` is not edited.
9. **Case C runs the step on CODEOWNERS.** `/backend/canary/internal_support` belongs to Internal
   Tools, but a later line gives `views/ai/ai_chat_*.py` to `pod-messaging`, and the last match
   wins. The `check-ignore` trick reports Internal Tools; an arm that uses it, or reads only the
   parent rule, fails this.
10. **Case D runs the step on the send-path list.** The file belongs to Arrivals-Departures, and it
    touches scheduled messages.

## Known failure modes (what the WITHOUT arm is expected to do)

- Runs constructive → adversarial → integrator and never reads `messaging-pr-review` (it has no way
  to know the file exists).
- Told about it, calls `Skill(messaging-pr-review)`, gets refused by
  `disable-model-invocation`, and drops the review.
- Follows the skill's own step 1: `gh pr checkout`, or diffs against
  `merge-base origin/master HEAD` — which on a stacked phase folds in other phases' commits.
- Follows its step-6 handoff to `messaging-pr-wrapup` and edits code outside the implementer.
- Treats "Needs human approval" as a BLOCK.
- Runs the Messaging step on Case B because the repo is canary.
- Skips Case B without naming its unowned files, or guesses an owner for them from the path.
- Resolves Case C's owner from the parent `internal_support` rule and skips it.
- Skips Case D because CODEOWNERS gives it to another team.

## Notes

The WITHOUT arm gets the pre-change `fledge-review` SKILL.md; the WITH arm gets the changed one.
Neither arm is told that `messaging-pr-review` exists — the task prompt doesn't name it — so the
WITHOUT arm measures "no rule" rather than "rule as already written". The arm still inherits the
session's global `CLAUDE.md`. For `fledge-lite`, run the same two cases against step 6 of its
SKILL.md, with two differences. Lite has no integrator, so criterion 3 becomes "its findings are
acted on or recorded in `REVIEW-CODE-final.md`". And Case A's send-time change is trip-wire 3 in
lite, so it never reaches step 6: use a Messaging-owned change off the sending pipeline instead,
such as the inbox UI (`frontend/hotels/src/chat/ChatConversations.vue`). Case D is trip-wire 3 in
lite for the same reason; drop it there.
