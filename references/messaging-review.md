# Messaging review — `REVIEW-CODE-2b-messaging.md`

The single source of truth for the optional review round that runs the Messaging team's own PR
review, `messaging-pr-review`, on a canary phase that changes Messaging code.
`skills/fledge-review/SKILL.md` (step 4b, code mode) and `skills/fledge-lite/SKILL.md` (step 6) run
it; `agents/fledge-reviewer-integrator.md` scores its output.

## Why the orchestrator runs it

`messaging-pr-review` sets `disable-model-invocation: true`, so the `Skill` tool refuses it, and
fledge's reviewer subagents cannot run skills at all. So the orchestrator **reads its `SKILL.md` and
follows the review steps itself**, inline — no spawn.

The Messaging team owns that file and revises it. Three rules follow:

- **Read it fresh on every run, and again every review cycle.** Never work from a version you
  remember from an earlier run or cycle.
- **Never copy its content into fledge** — not its rules, not its lists, not its categories into a
  fledge file. Point at it.
- **Never edit it**, and write nothing else under the canary repo during this step.

## Gate

Check these in order. The first one that fails is the skip reason.

1. **The project is the canary repo.** `git remote get-url origin` names
   `canary-technologies-corp/canary`.
2. **The file exists.** First `<repo root>/agent-plugins/claude-plugins/msg/skills/messaging-pr-review/SKILL.md`,
   then the newest installed copy:
   `"$(ls -d ~/.claude/plugins/cache/canary-plugins/msg/*/ | sort -V | tail -1)"skills/messaging-pr-review/SKILL.md`.
   Record which path you read.
3. **The change set touches Messaging code** — at least one changed file passes either test:
   - **CODEOWNERS gives it to `@canary-technologies-corp/pod-messaging`**, alone or with another
     team, by the last matching rule. Resolve owners with the recipe below. This catches Messaging
     code off the send path: chat, team chat, tickets, notifications.
   - **The send-path list in the `SKILL.md` you just read** (the trigger list of its bowerbird
     pass) matches the file's path or its lines in the phase diff (`REVIEW-PACKAGE.patch`, or
     `git diff <BASE> <HEAD>` when the patch is missing). This catches another team's file, or an
     unowned one, that touches the send path.

### Resolving owners

Run this from the repo root. It follows GitHub's documented CODEOWNERS rules: the last matching
line wins, and a line with no owners leaves the file unowned.

```bash
CO=$(ls .github/CODEOWNERS CODEOWNERS docs/CODEOWNERS 2>/dev/null | head -1)
git -c core.quotePath=false diff --name-only "$BASE" "$HEAD" > <phase-dir>/.changed-files
python3 -I - "$CO" <phase-dir>/.changed-files <<'EOF'
import re, sys
def rx(p):
    d = p.endswith('/'); p = p.rstrip('/')
    a = '/' in p; p = p.lstrip('/')
    out, i = '', 0
    while i < len(p):
        if p.startswith('**/', i): out += '(?:.*/)?'; i += 3
        elif p.startswith('**', i): out += '.*'; i += 2
        elif p[i] == '*': out += '[^/]*'; i += 1
        elif p[i] == '?': out += '[^/]'; i += 1
        else: out += re.escape(p[i]); i += 1
    tail = '' if p.endswith('/*') else ('/.*' if d else '(?:/.*)?')
    return re.compile(('^' if a else '^(?:.*/)?') + out + tail + '$')
rules = [(rx(f[0]), ' '.join(f[1:])) for f in
         (l.split() for l in open(sys.argv[1]) if l.strip() and not l.lstrip().startswith('#'))]
for f in open(sys.argv[2]).read().splitlines():
    print(f + '\t' + (next((o for r, o in reversed(rules) if r.match(f)), '') or 'UNOWNED'))
EOF
```

**Not `git check-ignore`.** CODEOWNERS uses gitignore's pattern syntax, but git stops at the first
ignored parent directory, so a nested rule never wins. `/backend/canary/internal_support` belongs to
Internal Tools, and a later line gives `views/ai/ai_chat_*.py` under it to Messaging;
`check-ignore` reports Internal Tools.

No CODEOWNERS file → every file counts as unowned, and the send-path test decides.

### Skipping

**Any gate fails → skip, and print one line naming it:**

> Messaging review: skipped — <not the canary repo | messaging-pr-review SKILL.md not found at the repo path or in the plugin cache | no Messaging-owned or send-path files>

When gate 3 is the one that fails and any changed file is unowned, append
` (unowned: <path>, <path>, …)` to the line. An unowned file may be Messaging code that CODEOWNERS
doesn't cover yet, so name it for the user to judge. Don't guess its owner from its path.

Pass that line to whoever consumes the round (the integrator, or lite's `REVIEW-CODE-final.md`), so
the skip is on record and not just in the transcript.

## Running it

Follow its review steps in order, as written, with these overrides and no others:

- **The diff is the phase's, not its own.** Replace its snapshot step with the range from
  `REVIEW-PACKAGE.md`: `git diff <BASE> <HEAD>` (or read `REVIEW-PACKAGE.patch`) and
  `git diff --stat <BASE> <HEAD>`. Never `gh pr checkout`, never `merge-base origin/master` — on a
  stacked phase that range folds in other phases' commits, and it breaks the same-package-every-run
  rule in `review-package-format.md`. This is its local-branch mode, pinned to fledge's range.
- **Review only.** Skip its handoff to `messaging-pr-wrapup`. Apply no fixes, run no lints or tests
  in this step: the implementer (full pipeline) or lite's step 6 acts on the findings.
- **Write only the output file.**

When a step it names can't run here (a reference it points at isn't installed, say), write that in
the output. Do not report a step as done that you did not do.

## Output

Write `<phase-dir>/REVIEW-CODE-2b-messaging.md` in its output shape, without the handoff section,
under this header:

```markdown
# Messaging review — <ISO timestamp>

- **Source:** <path of the SKILL.md read>
- **Diff range:** `<BASE>..<HEAD>` (from REVIEW-PACKAGE.md)
- **Why it ran:** <each file that passed gate 3, and which test: CODEOWNERS or send-path>
- **Unowned files:** <paths, or none>
- **Review cycle:** <N>
```

On a BLOCK it is archived with the cycle's other reviews and re-run on the next cycle's package.

## Scoring

The integrator (or lite's step 6, which has no integrator) maps each finding:

| Its category | In fledge |
|---|---|
| bug | consequential. Like any prior round's finding, it can be rejected only by showing it is wrong — cite the code, record it in `Reviewer notes`. Never downgraded to a suggestion or nit. |
| suggestion | scored with `severity-rubric.md` like any other finding |
| question | `Reviewer notes`, not a finding |
| nit | non-blocking nit |

Its labels don't lower fledge's own standards. A comment it calls a nit is still subject to the
comment-altitude check (C10, `comment-pass.md`); if that check fails, the integrator raises it as its
own finding, scored by the rubric, and the Messaging nit stays non-blocking.

Its risk tier and **approvability verdict are for the user**. Record them in `Reviewer notes`,
citing `REVIEW-CODE-2b-messaging.md`. They never set PASS, BLOCK or ESCALATE: "Needs human approval"
is not a BLOCK, and "Approvable" does not clear a consequential finding from another round.

Deduplicate against the other rounds as usual, crediting each round that raised a finding.
