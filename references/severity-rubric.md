# Severity rubric (hybrid)

Every review finding must carry **two** fields:

## 1. Severity
- **critical** — data loss, security vulnerability, production outage risk, contract violation of source-of-truth doc
- **major** — user-visible bug, regression, material deviation from source-of-truth, violation of project `CLAUDE.md` standards
- **minor** — non-user-visible correctness issue, maintainability problem with concrete downstream cost, test gap on a significant branch
- **nit** — style, naming, wording, micro-refactor with no material impact

## 2. Consequential (yes/no)
Answer yes if **any** of:
- Would cause a user-visible bug, regression, data loss, or security issue at rollout
- Violates a requirement in the source-of-truth doc
- Violates an explicit rule in the project's `CLAUDE.md` (or linked docs), or in the user's
  `~/.claude/shared/working-agreements.md` — the reviewers read both, so both bind
- Blocks another phase listed in the plan

Otherwise no.

## Blocking rule
A finding blocks iteration **iff** `consequential = yes`.

`critical` and `major` findings with `consequential = no` are extremely rare and require an explicit justification line. If you can't justify it, it's consequential.

`nit` findings are **always** non-blocking but should still be recorded — the implementer may address them opportunistically.

## Carve-outs: findings that look like style and are not

Two checklist items produce findings whose *form* is prose, so they get scored on their form
rather than their effect and land in `nit`, which is non-blocking and addressed
"opportunistically" — in practice, never. Both are scored explicitly below. They divide cleanly
and must not be cited together for the same line:

- **C10 governs text a developer reads** — comments and docstrings.
- **C11 governs text a user reads** — tooltips, labels, empty states, toasts, `aria-label`s,
  i18n values.

A docstring that contradicts the new behavior is **C10**, not C11, even though C11's grep list
finds it. One owner per line; the C8/C10 overlap is what happens otherwise.

### C10 — comment altitude

A comment finding looks like style — wording, verbosity — and the word "nit" sends it to a list
the implementer addresses "opportunistically", which in practice means never. That routing is
why comment altitude became the most-repeated review comment of the June–September 2026 window
while being written down the whole time. So it is scored explicitly:

| Shape | Severity | Consequential |
|---|---|---|
| Comment contradicts the code as it stands today | major | yes |
| Pins current behavior (field values, payload shape, counts, timings) | minor | yes |
| Restates the code, narrates the change, essay-length rationale | minor | yes |
| Bare `TODO:` / `FIXME:` with no ticket id | minor | yes |
| Commented-out code | minor | yes |
| Wording or phrasing of a comment that is correctly *kept* | nit | no |

The first row is wrong information a reader will act on. The rest are consequential because
they violate a standing rule in the working agreements and because **the fix is a deletion** —
there is no cost argument for deferring a change that removes a line. A round that blocks only
on comment findings is a cheap round, and it is the round that stops the habit.

This is a deliberate tightening and it is a dial. If comment findings start consuming review
cycles on their own, the thing to fix is the implementer's comment pass running late or not at
all — not this rubric.

### C11 — user-facing copy follows behaviour

C10's reader is a colleague who can open the code and discover the comment is lying. C11's
reader is a user who cannot, and who has no way to find out that the tooltip is wrong. So the
same contradiction is scored at least as hard here, not more softly:

| Shape | Severity | Consequential |
|---|---|---|
| User-facing string contradicts the behaviour it describes | major | yes |
| Same, on an `aria-label` or other assistive-only text | major | yes |
| String is no longer wrong but is now incomplete — the rule widened and the copy didn't | minor | yes |
| Default/source i18n value contradicts behaviour | major | yes |
| Re-translation of the other locales, once the source value is fixed | minor | no |
| Wording or tone of copy that is *correct* | nit | no |

`aria-label` gets its own row because it is the one shape where a reader has no fallback: a
sighted user can see that the button does something other than what the tooltip claims, and
correct for it. A screen-reader user is told what the control does and has nothing to check it
against. "It's only the aria-label" is backwards.

As with C10, the cost argument runs out: the fix is editing a string — no logic change, no test
churn, no migration. **The one real exception is i18n**, which is why it is two rows. Correcting
the default/source value blocks, because that is the string the behaviour contradicts. Getting
N locales re-translated is a tracked follow-up and does **not** block the round — but it is a
finding, so it gets written down rather than discovered by a user in a locale nobody reads.

Not C11's: the **PR description** against the final diff. C11's grep list includes it because
that is when you notice, but the description is not copy a user reads — it stays with C8 and
stays non-blocking.

## Format (for REVIEW-*.md files)

```markdown
### Finding N: <short title>
- **Severity:** major
- **Consequential:** yes — violates source-of-truth requirement "email must be verified before send"
- **Location:** `backend/myapp/email/sender.py:142`
- **Issue:** <one paragraph>
- **Suggested fix:** <one paragraph or code block>
```
