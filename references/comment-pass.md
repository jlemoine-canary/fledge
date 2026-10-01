# The comment pass

One procedure, run twice: by `fledge-implementer` as the last step before writing
`IMPLEMENTATION.md`, and by the reviewers as checklist item **C10**. This file is the
source of truth for both — the implementer executes it, the reviewer checks that it was
executed.

## Why this is a pass and not a disposition

"Default to zero comments" has been written down for a long time and comments keep
shipping anyway. A disposition isn't executed; a list is. So the pass is mechanical:
**enumerate the comments the change adds, then adjudicate each one individually.** A
comment survives only by naming the exception it falls under. Silence is deletion.

The asymmetry that justifies the effort: a missing comment costs a reader one minute of
reading the code. A wrong comment costs a reader an afternoon and a bug, because they
believed it. Comments are the only part of a change that nothing verifies — not the type
checker, not the tests, not CI. The pass is the only check there is.

## The procedure

1. **List them.** From the change's own diff, not from memory:

   ```bash
   BASE=$(cat .fledge/phases/<id>/.base-commit)
   git diff "$BASE" HEAD -U0 | grep -nE '^\+\s*(#|//|/\*|\*|<!--)'
   ```

   (`.base-commit` is the phase fork point `/fledge:fledge-implement` records at branch
   creation — the same base the review package will use, so your list and the reviewer's
   list are the same list.)

   Add docstrings and block comments the regex misses on the languages in the diff. If the
   list is empty, the pass is done — say so.

2. **Adjudicate each one.** For every line in the list, write a verdict: **keep** plus the
   exception it falls under, or **delete**. There is no third verdict. "Harmless" is a
   delete — a comment that is merely harmless has not met the burden.

3. **Apply the deletions**, then re-read each survivor against the final code one more time:
   a comment that was true when written may have stopped being true three iterations ago.

4. **Record the counts** in `IMPLEMENTATION.md` — comments added, kept, deleted, and one
   line per survivor naming its exception. That line is what the reviewer checks against the
   diff; a bare tick is not evidence.

## The keep list (closed)

These four are the whole list. A comment that is not one of them goes.

- **A non-obvious bound, constant, or ordering** — *why this number, why this sequence*.
  `# 200 is the PMS page cap — above it the provider truncates silently instead of erroring`
- **The invariant that makes an apparently-unsafe line safe** — the reason the `[0]`, the
  cast, or the missing guard is correct. These exist to stop a future reader "fixing" it.
- **A deliberate deviation, as `TODO(TICKET-ID):` or `FIXME(TICKET-ID):`.** A future-work
  note without a ticket id is not in the keep list — it is a delete. See the working
  agreements: a callout with no ticket is not acceptable.
- **An external constraint the code cannot express** — a spec clause, a vendor bug, a
  protocol quirk, a regulatory requirement. Link or name the source.

Not comments for the purpose of this pass, and never deleted by it: machine-read directives
(`# noqa`, `# type: ignore`, `eslint-disable`, pragmas), generated-file headers, licence
headers, and docstrings the project's own convention or tooling requires. A *required*
docstring is in scope only for its content — one that restates the signature is still a
finding, and one that describes behavior the diff just changed is a C11 defect.

## Delete on sight

- **Restating the code.** If the line below says what the comment says, the comment is
  duplication with no type checker holding the two halves together.
  `# Returns the chain for the reservation` above `def get_chain(reservation)`.
- **Narrating the change.** "moved here from X", "updated to fix the null case", "the
  deleted `ChannelTabsRow` used to own this". This is commit-message and PR-description
  content. In the source it goes stale the moment anything moves, and it reads as history
  to someone who never saw the before state.
- **Design rationale at essay length.** Good PR-description material, wrong home. Keep the
  one sentence naming the constraint; move the rest.
- **Anything that pins current behavior.** The sharpest shape, and the one the other rules
  miss, because it reads as documentation rather than as noise:
  - a field's current values — `# status is one of CONFIRMED, CHECKED_IN, CHECKED_OUT`
  - a payload or response shape — `# serializer emits {"id": int, "legs": [int]}`
  - what some *other* file does — `# the scheduler calls this twice per pass`
  - a count, a timing, or a measured number — `# ~40ms`, `# two call sites`

  **The test:** *could this line become false without anyone editing it?* If yes, it is
  rot-prone, and it will rot silently — nothing fails when the enum gains a member or the
  serializer gains a field. It then actively misleads, which is strictly worse than the
  comment never having been there.
- **Commented-out code**, in any quantity, for any reason. Version control has it.

## Rationalizations to reject

- *"It's documentation — it helps the next person."* It helps until it is wrong, and then
  it hurts more than it ever helped. If the shape is worth documenting, the type, the
  serializer, or the test documents it, and those are checked.
- *"It's accurate right now."* Every rotted comment was accurate once. Accuracy today is
  not the bar; being unable to go stale is.
- *"I'll keep it short instead of deleting it."* A shorter wrong comment is still wrong.
  The verdicts are keep and delete.
- *"The reviewer can decide."* The reviewer is the backstop, not the pass. Arriving at
  review with un-adjudicated comments is the failure this file exists to prevent.
- *"Deleting it loses the reasoning."* The reasoning goes in the PR description, where it
  is read once, at the moment it is relevant, by someone who wants it.
