# Scenario: fledge-lite / review-package-empty

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-lite` — whether the change set handed to its **single** review round actually contains
the change.

This fixture is **mechanical, not behavioural.** The other two `fledge-lite` fixtures score a
subagent's judgement; this one checks a determinate command result, so it needs no WITH/WITHOUT
arms and no scoring. `skills/fledge-writing-skills/SKILL.md` permits this form of RED explicitly:
"do it by hand and record the transcript excerpt." Running it as an A/B would be the wrong
instrument — the failure is not a misjudgement an agent might avoid on a lucky sample, it is
`git diff` returning nothing every time.

Lite has one review round and no integrator. An empty package is therefore not a degraded
review, it is **no review at all**, and nothing downstream would notice.

## Task prompt

> Follow `skills/fledge-lite/SKILL.md` steps 1 through 6 on any single-file ticket, in a fresh
> worktree, stopping the moment `REVIEW-PACKAGE.md` has been written. Do not spawn the reviewer.
>
> Then print, verbatim: the resolved `BASE` and `HEAD`, the output of `git log --oneline BASE..HEAD`,
> the output of `git diff --stat BASE HEAD`, the byte size of `REVIEW-PACKAGE.patch`, and the
> output of `git status --short`.

## Setup / context

Any ticket whose fix is one or two files. The defect does not depend on the ticket — it depends
only on the order of lite's own steps.

## Pass criteria (observable)

1. **`.fledge/phases/<id>/.base-commit` exists** by the end of step 1. Both
   `references/review-package-format.md` and `references/comment-pass.md` read this path;
   in the full pipeline `/fledge:fledge-implement` writes it.
2. **`git log --oneline BASE..HEAD` is non-empty** at step 6 — it names at least the commit
   carrying the fix.
3. **`git diff --stat BASE HEAD` lists every file the fix touched**, and
   `REVIEW-PACKAGE.patch` is larger than zero bytes.
4. **`git status --short` is clean** at step 6 — no part of the change is sitting only in the
   working tree, unreachable from the diff range.
5. **`BASE=$(cat .fledge/phases/<id>/.base-commit)` in `comment-pass.md` step 1 succeeds**
   rather than exiting non-zero with "No such file or directory".

## Known failure modes

What the skill does without the fix:

- Lite's only commit is **step 8**, two steps *after* the review package is compiled at step 6,
  so `BASE == HEAD` and every git command in the recipe returns empty.
- `.base-commit` is never written by lite at all — it is written by `/fledge:fledge-implement`,
  which lite does not run. `review-package-format.md` survives this on its
  `|| git merge-base` fallback; `comment-pass.md` has no fallback and fails outright.
- The skill's `## Error handling` entry for "the reviewer returns nothing" blames the briefing
  ("check you briefed it as the only round and gave it the diff"), which sends the next person
  looking in the wrong place.
