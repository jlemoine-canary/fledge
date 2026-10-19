# Eval result: fledge-lite / review-package-empty — 2026-10-19 (GREEN)

- **Scenario:** `evals/fledge-lite/review-package-empty/scenario.md`
- **Mode:** mechanical, single arm — same instrument as the RED run.
- **Version under test:** 0.9.1 (this branch)
- **Replayed against:** the same MSG-6641 worktree the RED run was captured in, so the only
  variable is the instructions.

## Scorecard

| Pass criterion | 0.9.0 (RED) | 0.9.1 (GREEN) |
|---|---|---|
| 1. `.base-commit` exists after step 1 | fail | **pass** — `48015e5b09a0…` |
| 2. `git log --oneline BASE..HEAD` non-empty | fail | **pass** — names the step-5 commit |
| 3. `git diff --stat` lists the touched files; patch > 0 bytes | fail | **pass** — 2 files, +132/−4, 5113-byte patch |
| 4. `git status --short` clean at step 6 | fail | **pass** — clean |
| 5. `comment-pass.md` step 1 `BASE=` succeeds | fail (exit 1) | **pass** — exit 0 |

0/5 → **5/5**. The comment-pass enumeration, which could not run at all under 0.9.0, now resolves
and finds 7 comment lines in the diff.

## What changed

Six edits in `skills/fledge-lite/SKILL.md`, two in `references/comment-pass.md`:

- **Step 1 records `.base-commit`.** The full pipeline gets this from
  `/fledge:fledge-implement`; lite does not run it, so lite writes it at pre-flight — and it
  must be pinned there, before `origin/<default>` moves.
- **Step 5 ends in a commit.** Not a second commit: step 8 now `--amend`s into it, so the branch
  still ends with one signed commit. The commit exists so that step 6 has a range to diff.
- **Step 6 asserts the range is non-empty** before spawning anything. This is the
  watched-it-fire check — the failure's whole character is that every command *succeeds* and
  prints nothing, so only an explicit assertion catches it.
- **`## Error handling` names the empty range first**, ahead of the briefing. The old entry sent
  the reader to the one place that was fine.
- **`comment-pass.md` gets the `|| git merge-base` fallback** that `review-package-format.md`
  already had — the two now resolve to the same base by construction rather than by luck — and
  names `.fledge/phases/<id>/COMMENT-PASS.md` as the small path's recording location, since lite
  produces no `IMPLEMENTATION.md`.

## REFACTOR — the loophole this fix opens

A fix that says "commit first" invites: *"the diff is right there in my working tree, the reviewer
can just read the files."* Added to `## Rationalizations to reject`. It is wrong twice: the
reviewer is handed `REVIEW-PACKAGE.md`, not a working tree, and pointing it at uncommitted files
abandons the determinism rule the package format exists to enforce.

## Also fixed, same run, different defect

Step 4 demanded that *every* test fail for the right reason. MSG-6641's acceptance criteria
included "desktop browsers still show notifications as they do today" — coverable only by tests
that characterise working behaviour, which are green by construction. Three of that run's four
tests were. Read literally, step 4 forced dropping them (losing an acceptance criterion) or
contriving them red (worse).

Step 4 now sorts tests into **defect tests** (must be red for the right reason; at least one
required) and **guard tests** (green from the start, and correct to be), and names step 7 as what
gives the guards teeth. That is not a weakening: on MSG-6641 the mutation check caught all five
mutations, two of which had survived green until the reviewer's findings were fixed.

No RED fixture is filed for this one — it is a documentation contradiction, not a behaviour the
harness can score, and the MSG-6641 run is the record.

## Verdict

GREEN, 5/5. Not yet released: per `README.md` § Releasing changes this is step 2 of 6, and the
release is unverified until a real small-ticket run produces a `REVIEW-PACKAGE.md` with a
non-empty `## Commits` section.
