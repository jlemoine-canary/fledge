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

## Carve-out: comment altitude (C10) is not a nit

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

The first row is a defect on the same footing as C11: wrong information a reader will act on.
The rest are consequential because they violate a standing rule in the working agreements and
because **the fix is a deletion** — there is no cost argument for deferring a change that
removes a line. A round that blocks only on comment findings is a cheap round, and it is the
round that stops the habit.

This is a deliberate tightening and it is a dial. If comment findings start consuming review
cycles on their own, the thing to fix is the implementer's comment pass running late or not at
all — not this rubric.

## Format (for REVIEW-*.md files)

```markdown
### Finding N: <short title>
- **Severity:** major
- **Consequential:** yes — violates source-of-truth requirement "email must be verified before send"
- **Location:** `backend/myapp/email/sender.py:142`
- **Issue:** <one paragraph>
- **Suggested fix:** <one paragraph or code block>
```
