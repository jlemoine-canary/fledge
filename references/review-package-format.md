# Review package format — `REVIEW-PACKAGE.md`

The deterministic change-set bundle the code-mode reviewers consume. This is the single
source of truth for how a phase's change set is compiled. `skills/fledge-review/SKILL.md`
(step 2) produces it; `agents/fledge-reviewer-constructive.md`,
`agents/fledge-reviewer-adversarial.md`, and `agents/fledge-reviewer-integrator.md` consume
it (their `code`-mode artifact path is this file).

It supersedes the earlier ad-hoc `CHANGES.md`. The point is **reproducibility**: the same
base/head must yield the same package, regardless of which run or which agent compiles it.
Two reviewers — or the same phase across two review cycles — must not see two differently
shaped change sets.

## The determinism rule

The package is **mechanically derived** from a fixed sequence of git commands against a
fixed base/head. The compiler runs the recipe and transcribes the output — it does **not**
editorialize. No "review focus" prose, no subjective "notes" column, no judgement about what
matters. That is the reviewers' job; baking it into the package biases every persona
identically and is the exact thing that makes hand-assembled change sets diverge run to run.

Same base + head → byte-identical package, modulo the generated-at timestamp.

## Base/head resolution (deterministic, in order)

- **head** = `git rev-parse HEAD` — the phase branch tip.
- **base**, in priority order:
  1. The recorded phase base: contents of `.fledge/phases/<id>/.base-commit` if it exists
     (written by `/fledge:fledge-implement` when it cuts the phase branch). This is the
     authoritative fork point and survives `main` advancing or multiple phases sharing
     history.
  2. Else `git merge-base HEAD <default-branch>`, where `<default-branch>` comes from
     `git symbolic-ref --short refs/remotes/origin/HEAD` (e.g. `origin/main`).

Never diff against `HEAD~N`, against the moving `origin/<default>` tip directly, or against
"latest" — all three are non-reproducible (they fold in upstream commits or drop phase
commits). Always pin the resolved SHAs into the package so the range is exact.

## The recipe (run verbatim, in this order)

```bash
BASE=$(cat .fledge/phases/<id>/.base-commit 2>/dev/null \
       || git merge-base HEAD "$(git symbolic-ref --short refs/remotes/origin/HEAD)")
HEAD=$(git rev-parse HEAD)

git log --oneline "$BASE".."$HEAD"        # → Commits section
git diff --stat "$BASE" "$HEAD"           # → Stats section
git diff --name-status "$BASE" "$HEAD"    # → Files section (status + path, git's order)
git diff "$BASE" "$HEAD" > .fledge/phases/<id>/REVIEW-PACKAGE.patch   # full diff, saved
```

The full diff is **saved to a sibling patch file**, not inlined into `REVIEW-PACKAGE.md`.
Reviewers receive the patch path and read it on demand (context-budget protocol: paths, not
inline bodies). This resolves the ambiguity that makes hand-built change sets diverge — one
run inlines the diff, another omits it, a third re-runs git itself.

## Output structure (fixed sections, fixed order)

Write `.fledge/phases/<id>/REVIEW-PACKAGE.md` with exactly these sections, in this order:

```markdown
# Review package — Phase <id>

- **Generated:** <ISO timestamp> (code-review cycle <N>)
- **Base:** `<BASE sha>` (<source: recorded .base-commit | merge-base with origin/main>)
- **Head:** `<HEAD sha>`
- **Diff range:** `<BASE>..<HEAD>`
- **Full diff:** `REVIEW-PACKAGE.patch` (read on demand)

## Commits
<verbatim `git log --oneline BASE..HEAD` output, fenced>

## Files
<verbatim `git diff --name-status BASE HEAD` — status letter + path, git's ordering preserved>

## Stats
<verbatim `git diff --stat BASE HEAD` output, fenced>

## Area split
<One line. Default: "None — N files, single review pass." If >10 files OR the change spans
multiple surfaces (see references/qa-by-surface.md roots: frontend/ vs backend/), state the
split the reviewers should use, grouped by top-level path prefix. Factual grouping only —
no opinion on which area is risky.>
```

Do **not** add sections beyond these. The `--name-status` status letters (M/A/D/R) and git's
file ordering are preserved verbatim — do not re-sort, re-group, or annotate individual files.

## What this package does NOT contain
- No editorial "review focus" / "watch out for" prose — that is the reviewers' output, not
  the package's input.
- No per-file subjective notes column.
- No inline diff hunks — those live in `REVIEW-PACKAGE.patch`.
- No SoT comparison — reviewers do that against the cycle's `.sot-snapshot.md`.
