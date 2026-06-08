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
- Violates an explicit rule in the project's `CLAUDE.md` (or linked docs)
- Blocks another phase listed in the plan

Otherwise no.

## Blocking rule
A finding blocks iteration **iff** `consequential = yes`.

`critical` and `major` findings with `consequential = no` are extremely rare and require an explicit justification line. If you can't justify it, it's consequential.

`nit` findings are **always** non-blocking but should still be recorded — the implementer may address them opportunistically.

## Format (for REVIEW-*.md files)

```markdown
### Finding N: <short title>
- **Severity:** major
- **Consequential:** yes — violates source-of-truth requirement "email must be verified before send"
- **Location:** `backend/myapp/email/sender.py:142`
- **Issue:** <one paragraph>
- **Suggested fix:** <one paragraph or code block>
```
