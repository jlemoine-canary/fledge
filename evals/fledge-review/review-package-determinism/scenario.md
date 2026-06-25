# Scenario: fledge-review / review-package-determinism

> Fixture for `/fledge:fledge-eval`. Exercises the deterministic review-package handoff:
> compiling the code-mode change set must follow a fixed recipe so two independent runs
> produce the same package, instead of each agent reconstructing its own ad-hoc view.

## Skill under test
`fledge-review` (code mode, step 2 — compile the change set) deferring to
`references/review-package-format.md`. Given a phase and its git state, the produced
change-set artifact must be deterministic: same base/head → same package structure,
regardless of which run produced it.

## Task prompt
You are the orchestrator running the code-review stage for a fledge phase. You need to
compile the change set that three reviewer subagents will consume. The phase is
`02-booking-pricing` at `.fledge/phases/02-booking-pricing/`. Its branch was cut from
`origin/main`.

Here is the current git state (canned for this exercise — treat these as the real command
outputs you would get):

```
$ git symbolic-ref --short refs/remotes/origin/HEAD
origin/main

$ git merge-base HEAD origin/main
a1b2c3d

$ git rev-parse HEAD
f9e8d7c

$ git log --oneline a1b2c3d..f9e8d7c
f9e8d7c feat(bookings): apply loyalty discount before tax
3c4d5e6 test(bookings): pricing order failing tests

$ git diff --name-status a1b2c3d f9e8d7c
M       backend/bookings/pricing.py
A       backend/bookings/tests/test_pricing.py
M       backend/bookings/models.py

$ git diff --stat a1b2c3d f9e8d7c
 backend/bookings/pricing.py            | 24 ++++++++++++++----
 backend/bookings/tests/test_pricing.py | 38 ++++++++++++++++++++++++++
 backend/bookings/models.py             |  3 +-
 3 files changed, 62 insertions(+), 5 deletions(-)
```

Produce the change-set artifact the reviewers will read. State where you write it, its
exact section structure, and what it contains. Show the artifact.

## Setup / context
- The phase directory exists; reviewers are spawned with a path to the artifact you produce.
- Both arms get exactly the canned git state above. No real repo access is needed — the
  exercise is about the *shape and determinism* of the artifact, not running git.
- Reviewers in this pipeline receive paths, never inline content.

## Pass criteria (observable)
1. The artifact resolves its **base commit by a stated deterministic rule** (the recorded
   phase base commit, else `git merge-base HEAD <default-branch>`) — it names the base
   (`a1b2c3d`) and head (`f9e8d7c`), not a guessed `HEAD~1` / `main` / "latest".
2. The artifact contains a **fixed, ordered set of sections**: metadata (phase, base, head),
   the commit list, the file table from `--name-status` (status + path), and the diff (or a
   pointer to a saved patch) — present and in that order.
3. The **file list preserves git's `--name-status` ordering and status letters** (M/A/D),
   not a re-sorted or prose-summarized list that drops the status column.
4. A **second independent run of this same task produces a structurally identical artifact**
   — same filename, same sections in the same order, same base/head — differing only in
   incidental wording, not structure. (Run the arm ≥2×; compare.)

## Known failure modes (what the WITHOUT arm is expected to do)
- Each run picks a different base ("I'll diff against `HEAD~1`" vs "against `main`" vs the
  merge-base) — non-reproducible.
- Free-form prose summary of "what changed" instead of a fixed structure; the `--name-status`
  status letters get dropped or files get re-sorted/grouped differently each run.
- Omits the commit list and/or the diff in one run but includes it in another.
- Writes to an ad-hoc filename (`CHANGES.md`, `diff.md`, `review-notes.md`) that varies by run.

## Notes
- Subagent output is stochastic; the *divergence* between runs is the failure being measured,
  so run the WITHOUT arm ≥2× and compare the two artifacts structurally. The WITH arm should
  converge: ≥2 runs yield the same structure.
