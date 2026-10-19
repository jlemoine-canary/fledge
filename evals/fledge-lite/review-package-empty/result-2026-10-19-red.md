# Eval result: fledge-lite / review-package-empty — 2026-10-19 (RED)

- **Scenario:** `evals/fledge-lite/review-package-empty/scenario.md`
- **Mode:** mechanical, single arm. No WITH/WITHOUT split — see the scenario for why.
- **Version under test:** 0.9.0, installed cache at
  `~/.claude/plugins/cache/fledge/fledge/0.9.0` (`gitCommitSha` `e292a9a`)
- **Captured during:** the real 0.9.0 release-verification run on Linear **MSG-6641**
  (`frontend/packages/shared/utilities/notification.ts` in the Canary monorepo), not a sandbox.

**What this measures.** Whether the change set handed to lite's single review round contains the
change. It does not.

## Scorecard

| Pass criterion | 0.9.0 |
|---|---|
| 1. `.base-commit` exists after step 1 | **fail** — never written; lite does not run `/fledge:fledge-implement` |
| 2. `git log --oneline BASE..HEAD` non-empty | **fail** — empty |
| 3. `git diff --stat BASE HEAD` lists the touched files; patch > 0 bytes | **fail** — empty; patch would be 0 bytes |
| 4. `git status --short` clean at step 6 | **fail** — the entire change is uncommitted |
| 5. `comment-pass.md` step 1 `BASE=` succeeds | **fail** — exit 1, "No such file or directory" |

0 of 5.

## The transcript

`comment-pass.md` step 1, run verbatim:

```
$ BASE=$(cat .fledge/phases/MSG-6641/.base-commit)
cat: .fledge/phases/MSG-6641/.base-commit: No such file or directory
exit=1  BASE=[]
```

`review-package-format.md`'s recipe, run verbatim — it survives on its fallback, and that is
precisely what hides the defect:

```
BASE=48015e5b09a081958a3f50c020765db83a122fb3
HEAD=48015e5b09a081958a3f50c020765db83a122fb3
--- git log --oneline BASE..HEAD ---
(end of log)
--- git diff --stat BASE HEAD ---
(end of stat)

=== but the actual work is here: ===
 M frontend/packages/shared/utilities/notification.ts
?? frontend/packages/shared/utilities/notification.test.ts
```

`BASE == HEAD`. Both commands succeed and return nothing. No error is raised anywhere.

## Why this is blocking rather than a nit

The full pipeline runs six review rounds and an integrator; a thin package in one of them is
absorbed. Lite runs **one** round and deletes the integrator, and its own framing says so:
*"Nobody downstream will catch what you miss."* An empty package means the single reviewer is
handed a no-op and reports finding nothing — which is indistinguishable, from the orchestrator's
side, from a clean change.

The skill's `## Error handling` makes that worse by pointing at the wrong cause:

> **The reviewer returns nothing** → check you briefed it as the only round and gave it the diff.
> A one-round review that found nothing on a real fix is more likely under-briefed than clean.

The briefing is not the problem. The diff range is empty before the briefing is ever written.

## What the same run showed the fix is worth

The MSG-6641 run continued by committing before step 6 — the fix this result motivates. With a
commit in range the package compiled correctly (1 commit, 2 files, 148-line patch), and the single
adversarial round returned **10 findings, 2 of them major**, each backed by a mutation the
reviewer ran itself and watched survive:

- the acceptance criteria's load-bearing clause ("with nothing sent to Sentry") had no assertion,
  and `console.error` in the new catch shipped green — into an app whose Sentry is initialised
  with `captureConsoleIntegration({ levels: ["error"] })`;
- no test reproduced the production precondition, so the guard the ticket blames could be deleted
  with the suite still green.

Both would have shipped against the empty package. That is the measured cost of this defect: not
a weaker review, a missing one.

## Verdict

RED confirmed, 0/5, from a real run rather than a constructed one. Fix and re-run.
