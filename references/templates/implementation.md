# Implementation — <phase-id>

## Files changed
| File | Action | Lines (±) |
|---|---|---|
| backend/myapp/x/y.py | created | +120 |
| backend/myapp/x/tests/test_y.py | created | +85 |
| frontend/src/components/Z.vue | edited | +12 / -4 |

Total: <N> files, +<X> / -<Y> lines.

## Deviations from PLAN.md
<List any deviation, with justification. If none, write "None.">

- <deviation> — <why>

## Findings deferred
<Anything from a prior review that wasn't addressed in this pass and why.>

- <finding> — <why deferred or punted>

## Tests passing
- `<test command 1>` — <N> tests pass
- `<test command 2>` — <N> tests pass

## Lint / typecheck status
- `<command>` — clean
- `<command>` — clean

## Comment pass
<Per `references/comment-pass.md`. Counts, then one line per survivor naming its exception.
A tick with no survivor lines is not evidence — if the diff added no comments, write
"0 added" and move on.>

Comments added: <N> · kept: <K> · deleted: <N-K>

| Location | Verdict | Exception |
|---|---|---|
| `backend/myapp/x/y.py:88` | keep | non-obvious bound — provider truncates silently above it |
| `backend/myapp/x/y.py:140` | keep | invariant making the `[0]` safe |

## Cleanup pass
- [x] No committed screenshots or scratch files
- [x] No debug logging beyond what the plan specified
- [x] `git diff --stat` matches the plan's File plan (deviations noted above)
- [x] This description matches the actual diff
