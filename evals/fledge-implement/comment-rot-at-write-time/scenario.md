# Scenario: fledge-implement / comment-rot-at-write-time

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-implement` / `agents/fledge-implementer.md` — whether the implementer **writes**
rot-prone comments in the first place, when the task description hands it four standing
invitations to do so.

Its sibling fixture `comment-altitude-pass` tests the *backstop*: given a file already full
of bad comments and an instruction to clean up, does the agent adjudicate them? A capable
agent passes that one unaided, because the prompt itself cues comment review. **This fixture
tests the habit instead.** The arm is asked to implement, not to clean up; the cleanup step is
one bullet among four, exactly as it sits in the real implementer. Nothing in the task says
the word "comment".

## Task prompt

> Implement `MessageThrottle.allow()` in a new file `backend/notifications/throttle.py` of a
> mature Django monorepo. Write the implementation only — no tests, and do not create, read or
> edit any files on disk; output the file inline.
>
> **Plan**
>
> - `MessageThrottle(hotel)` exposes `allow(channel) -> dict`.
> - It returns `{"allowed": bool, "retry_after": int | None}` — `retry_after` is seconds, and
>   is `None` whenever `allowed` is `True`.
> - `channel` is one of the values of the existing `NotifiableChannel` enum: `EMAIL`, `SMS`,
>   `WHATSAPP`, `PUSH`.
> - Counting is via the existing `redis_client` with key `throttle:{hotel.id}:{channel}` and a
>   rolling 60-second window. Assume `redis_client.incr(key)` and `redis_client.expire(key, n)`.
> - The global cap is 100 messages per window for every channel.
> - When the cap is exceeded, back off — but **cap the backoff at 60 seconds**. The downstream
>   vendor replies `429` with no `Retry-After` header, and their published guidance is that the
>   window is always 60 seconds, so waiting longer than that just loses throughput.
> - Per-hotel overrides of the cap are planned for a later ticket. For now every hotel gets the
>   global cap.
> - This supersedes the throttling logic that currently lives in
>   `backend/notifications/legacy_throttle.py`, which will be deleted in a follow-up.
>
> When the implementation is done, do your standard cleanup pass before handing off: no scratch
> files, no debug logging beyond what the plan specified, no stale leftovers from earlier
> iterations, and confirm the change set matches the plan. Then write a short handoff summary.

## Setup / context

- Treat the repo as a mature Django monorepo reviewed by humans, with a project `CLAUDE.md`
  that asks for self-documenting code.
- The arm produces one file plus a short summary. No tests, no plan artifacts.

## Pass criteria (observable)

Read the produced file. Each criterion is a property of the committed code, not of the prose.

1. **No comment or docstring pins the returned dict's shape** — no restatement of
   `{"allowed": ..., "retry_after": ...}`, its key names, or its types. The signature, the
   return statement and the serializer are where that lives.
2. **No comment or docstring enumerates the `NotifiableChannel` values.** Naming the enum is
   fine; copying `EMAIL, SMS, WHATSAPP, PUSH` into prose is the finding — it becomes false
   when the enum gains a member and nothing fails when it does.
3. **No comment narrates the supersession** — no "replaces `legacy_throttle.py`", "moved from",
   "new implementation of". That is PR-description content.
4. **The 60-second backoff cap carries a comment naming the vendor constraint.** This is the
   one comment that must exist: the number is unguessable from the code, and an uncommented
   magic `60` is the opposite failure. Deleting all comments fails this criterion.
5. **The deferred per-hotel override is either absent or marked `TODO(TICKET-ID):`** — a bare
   `TODO:` or a prose comment about future work is a fail.
6. **No comment restates the line below it** (`# increment the counter` over `incr(...)`).

## Known failure modes (what the WITHOUT arm is expected to do)

- Writes a docstring for `allow()` that spells out the return dict's keys and types, because
  the return type is `dict` and documenting it feels like a kindness. This is the single most
  likely failure and the most expensive: it is the shape that rots silently.
- Copies the four channel values into a docstring or a comment near the signature.
- Writes `# Supersedes legacy_throttle.py` at the top of the file.
- Writes `# TODO: per-hotel overrides` with no ticket.
- Leaves the `60` bare, or comments it with `# cap at 60 seconds` — restating the code rather
  than naming the vendor constraint that makes 60 the right number.

## Notes

- The plan text deliberately supplies, in prose, exactly the four things that must *not* become
  comments, plus the one that must. An implementer that transcribes its brief into the source
  fails 1–3 and 5; one that transcribes nothing fails 4.
- Criterion 4 is what stops "write no comments at all" from passing.
- Stochastic, and generation varies more than adjudication. Run 2–3 times per arm before
  trusting a borderline result.
