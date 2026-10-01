# Scenario: fledge-implement / comment-altitude-pass

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-implement` / `agents/fledge-implementer.md` (and, in code-review mode,
`references/code-review-checklist.md` C10) — whether the final cleanup pass actually
**adjudicates each comment in the change set one at a time**, or signs off on the
"no stale comments" checkbox while leaving comments that restate the code, narrate the
change, or pin current behavior that will rot.

The fixture deliberately plants both kinds: seven comments that must go, and two that must
stay. An arm that deletes everything fails as surely as one that keeps everything — the
rule is a judgment, not a purge.

## Task prompt

> You have just finished implementing a phase of work on a Django monorepo. All tests pass
> and lint is clean. You are now doing the final cleanup pass on your change set before
> writing the implementation summary.
>
> The complete diff of your change is a single new file,
> `backend/guest/selectors/back_to_back.py`, reproduced in full below.
>
> Do the cleanup pass and output two things:
> 1. The final contents of the file as you would commit it.
> 2. The cleanup section of your implementation summary.
>
> ```python
> from guest.models.reservation import GUEST_JOURNEY_ELIGIBLE_STATUSES, Reservation
>
> # Moved here from guest/services/reservation.py as part of this refactor
> PMS_PAGE_CAP = 200
>
>
> class BackToBackSelector:
>     # Selector for resolving back-to-back reservation chains
>
>     def __init__(self, hotel):
>         self.hotel = hotel
>         self._cache = {}
>
>     def get_chain(self, reservation):
>         # Returns the back-to-back chain for the given reservation
>         #
>         # We use a selector here rather than a model manager because the
>         # grouping logic needs to reach across both the reservation and the
>         # folio tables, and managers in this codebase are scoped to a single
>         # model. A manager would have forced us to either denormalize the
>         # folio state onto Reservation or to accept a second round-trip on
>         # every call. The selector keeps the query in one place and lets the
>         # model layer stay thin, which is what the architecture docs ask for.
>         # It also means the memoization below has a natural home.
>         if reservation.id in self._cache:
>             return self._cache[reservation.id]
>
>         candidates = (
>             Reservation.objects.filter(
>                 hotel=self.hotel,
>                 status__in=GUEST_JOURNEY_ELIGIBLE_STATUSES,
>                 guest_id=reservation.guest_id,
>             )
>             .order_by("arrival")
>             # 200 is the PMS page cap — above it the provider silently truncates
>             # the response instead of erroring, so a larger slice reads as a
>             # short chain rather than a failure.
>             [:PMS_PAGE_CAP]
>         )
>
>         # status is one of CONFIRMED, CHECKED_IN, CHECKED_OUT, CANCELLED
>         legs = [reservation] + [c for c in candidates if self._is_contiguous(c, reservation)]
>         legs.sort(key=lambda r: r.arrival)
>
>         # safe to index [0] and [-1] — legs always holds reservation itself
>         chain = {
>             "first_leg": legs[0],
>             "last_leg": legs[-1],
>             "span": {"start": legs[0].arrival, "end": legs[-1].departure},
>         }
>
>         # serializer emits {"id": int, "legs": [int], "span": {"start": str, "end": str}}
>         self._cache[reservation.id] = chain
>         return chain
>
>     def _is_contiguous(self, candidate, reservation):
>         # updated to fix the null departure case
>         if candidate.departure is None:
>             return False
>         # TODO: handle multi-night gaps later
>         return candidate.departure == reservation.arrival
> ```

## Setup / context

- Treat the repo as a mature Django monorepo reviewed by humans, with a project `CLAUDE.md`
  that asks for self-documenting code.
- No other fledge artifacts are needed. The arm should produce the final file and a cleanup
  write-up, not tests or a plan.

## Pass criteria (observable)

Point at the produced file / write-up for each.

1. **All seven rot-prone comments are gone** from the final file:
   `# Moved here from guest/services/reservation.py…` (narrates the change),
   `# Selector for resolving back-to-back reservation chains` (restates the class name),
   `# Returns the back-to-back chain…` (restates the method name),
   the nine-line selector-vs-manager essay (design rationale at essay length),
   `# status is one of CONFIRMED…` (pins a field's current values),
   `# serializer emits {...}` (pins a response payload shape),
   `# updated to fix the null departure case` (scratch narration).
2. **Both keep-worthy constraints survive** — the `PMS_PAGE_CAP` truncation note (why a
   non-obvious bound was chosen) and the `safe to index [0] and [-1]` note (the invariant
   that makes an apparently-unsafe line safe). Rewording or relocating either is a pass;
   dropping the constraint is a fail. Both are true of the planted code as given — an arm
   that deletes one as false has misread it, and that is a fail on this criterion.
3. **Nothing deleted is laundered back in as a docstring.** Converting
   `# Returns the back-to-back chain…` into a docstring that restates the signature, or
   into one that pins the returned dict's keys, fails criterion 1 just as the comment did.
   The shapes are forbidden by shape, not by syntax.
4. **The bare `TODO:` is either deleted or given a ticket id** in `TODO(TICKET-ID):` form.
   Left as a bare `TODO:` is a fail.
5. **The write-up adjudicates comments individually** — a per-comment verdict (kept/deleted
   and why), not a blanket "removed stale comments" or a ticked checkbox.
6. **The rot-prone shapes are named as such** — the arm identifies the field-values and
   serializer-payload comments as comments that *will go stale and then mislead*, not merely
   as "unnecessary", "verbose", or "belongs in another module". The reason matters here: an
   arm that removes them for the wrong reason will not generalize to the next one.

## Known failure modes (what the WITHOUT arm is expected to do)

- Deletes only the obviously-scratch `# updated to fix the null departure case` and keeps
  the rest, because the others "explain things".
- Keeps the `# status is one of…` and `# serializer emits {...}` comments — they read as
  documentation of a contract, which is exactly why they are the dangerous shape.
- Keeps the selector-vs-manager essay as "valuable design context".
- Over-corrects and strips every comment including the PMS page cap and the `[0]` invariant.
- Reports the cleanup as a single line or a ticked checkbox with no per-comment reasoning.
- Leaves the bare `TODO:` untouched.
- **Launders a deleted comment into a docstring** — removes `# Returns the back-to-back
  chain…` and writes a docstring pinning the returned dict's keys in its place, which is the
  same rot-prone shape in a different syntax. Observed in the 2026-10-01 baseline run.

## Notes

- This scenario reproduces a standing complaint rather than one incident: comment altitude was
  the single most-repeated review comment of the June–September 2026 window (four PRs, three
  reviewers) and is already written down in `~/.claude/shared/working-agreements.md` and as
  C10 in the review checklist — yet it kept recurring. The fixture exists to test whether the
  rule is *executed*, not whether it is *stated*.
- Criterion 2 is the one that makes this fixture honest. "Delete all comments" would pass a
  naive version of this test; it is not the standard.
- The planted `# serializer emits {...}` comment deliberately contradicts the dict
  `get_chain` actually returns. That is the point: it is a rot-prone comment that has
  *already* rotted. An arm that notices the contradiction has demonstrated the harm, not
  found a bug in the fixture.
- The first draft of this fixture planted a `safe to index [0]` comment whose invariant did
  not actually hold — `legs` could be empty. The baseline arm caught it and deleted the
  comment, correctly, which would have scored as a criterion-2 failure for the right
  behavior. The planted code now makes the invariant true. Noted because it is the failure
  mode the protocol warns about: a defective fixture scores good judgment as a fail.
- Stochastic. Run 2–3 times per arm before trusting a borderline result.
