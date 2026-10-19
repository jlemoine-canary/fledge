---
name: fledge-lite
description: Use when the user hands over one small ticket — a single source of truth, one service, a localized fix — and says "fledge this", "just run it", "small one", or when /fledge:fledge's step-0 estimate is out of proportion to the change. Runs the trimmed pipeline — orchestrator-written plan, no planner or implementer subagent, one adversarial review round, a mandatory mutation check, one checkpoint — and stops and hands back to full /fledge:fledge the moment the work outgrows it.
---

# fledge-lite

The full pipeline spends ~9 subagents, two checkpoints and six review rounds. On an ~80-line
single-surface fix with one ticket behind it, most of that spend has nothing to work on: the
planner writes four bullets, the plan review reviews four bullets, and the integrator reconciles
two reviews that agree.

**Lite is a subtraction, not a second pipeline.** Every stage it drops is dropped *by name* in the
closing report (step 10), so lite runs and full runs stay comparable and the drops stay arguable.
No standard is lowered: the pre-flight, failing-tests-first, the reviewer's checklist, the comment
pass, lint/format/typecheck and signed commits all survive unchanged. One check gets **stronger** —
the mutation check moves from evidence a reviewer may demand to a step you run yourself, before
the reviewer sees the diff.

The small path's only real risk is being used where it doesn't belong. `## Escalation trip-wires`
is the whole defense, and it is the one section here that is never optional.

## Rationalizations to reject

Every one of these is the same move: you have already committed to the fast path and the trip-wire
is now inconvenient. The trip-wire is right and you are not.

- *"It trips one wire, but only barely — I'll be careful."* Carefulness is not a stage. A tripped
  wire means stop and ask, regardless of how close the call felt. Lite has one review round; the
  slack that would absorb your misjudgment is exactly what was removed.
- *"It's one guard plus a column."* A column is a migration, and a migration is never one unit of
  work — see the migration rule in `~/.claude/shared/working-agreements.md` (who owns the existing
  rows, which other query's plan changed, which sibling surfaces share the model) and its twin at
  checklist **P10**. It also forces a landing plan with at least two rows, which lite does not do.
- *"The plan grew to seven bullets, but three are trivial."* The bullet count is a proxy for how
  much the one reviewer has to hold at once. Trivial bullets still occupy it. Count them.
- *"Escalating now wastes the plan I just wrote."* It doesn't — hand the PLAN.md over as a seed
  (`/fledge:fledge --from=plan`). Escalation is a handoff, not a restart. And if the wire fired at
  step 3, there is no plan to waste: that is the check working at the cheapest possible moment.
- *"The reviewer's finding is blocking, but I can see the local fix."* If the fix is a deletion or
  a string edit, make it (step 6). If it needs a redesign, that is a trip-wire; a redesign reviewed
  by nobody is the failure mode lite is most exposed to.
- *"The diff is right there in my working tree — the reviewer can just read the files."* It
  cannot. It is given `REVIEW-PACKAGE.md`, and that file is compiled from a commit range. Pointing
  a reviewer at "the working tree" also abandons the determinism rule the package exists to
  enforce: same base and head, same package, every run. Commit, then review.
- *"I'm at 60% context but nearly done."* "Nearly done" is the estimate that was wrong when you
  entered lite. Escalate at 50%.
- *"I'll note the escalation in the final report instead of interrupting."* Escalation means stop
  and ask. A trip-wire reported after the fact is a trip-wire that did nothing.

## Preconditions

- Exactly **one** source of truth, and it is reachable (Linear, Notion or GitHub).
- The change is expected to be one service, one surface, and roughly ≤5 change bullets / ≤200
  changed lines. If you cannot estimate this yet, fetch the source first (step 2) and decide at
  step 3 — not now.
- No `.fledge/` or `.context/` state for this ticket from an earlier run.

## Arguments

- **Ticket id** (required) — e.g. `MSG-6639`. Also the phase id.
- **`--from=plan`** — a `PLAN.md` already exists for this ticket (e.g. you are re-entering after an
  escalation was resolved). Skip step 3's authoring, but **run its trip-wire check anyway.**

## Escalation trip-wires

Check this list at **step 3, before writing the plan**, and again **every time the plan changes** —
including when the reviewer's findings change it and when implementation reveals the shape was
wrong. If **any** item holds, stop.

1. More than one source of truth, or the sources conflict.
2. A migration, a model change, or anything the split-migration rule touches.
3. The message scheduling/sending pipeline, payments, auth/permissions, or idempotency — risk
   level 5.
4. More than one service, or both frontend and backend.
5. The plan grows past ~5 change bullets or ~200 changed lines.
6. A new API endpoint, Kafka event, or other public contract change.
7. User-facing copy or i18n strings that need re-translation.
8. The reviewer returns a blocking finding that needs a redesign rather than a local fix.
9. The tests cannot be made to fail for the right reason within 2 attempts.
10. Context use passes 50%.

**Escalation means stop and ask — never silently continue in lite mode.** Say which wire tripped
and what it implies, then offer the full pipeline. **Which offer depends on what exists:**

- **A `PLAN.md` exists** (the wire fired at step 5, 6 or later) → offer
  `/fledge:fledge --from=plan` with the lite `PLAN.md` as the seed. Say what is already done —
  the branch, the tests, the diff — so none of it is redone.
- **No `PLAN.md` yet** (the wire fired at the step-3 check, which is the common case, since that
  check runs *before* the plan is written) → offer plain `/fledge:fledge`. There is no seed. Do
  not invent one by writing a plan you have just decided you are not qualified to write; the
  fetched ticket and the pre-flight worktree carry over on their own.

Do not pre-emptively start the full pipeline either; the user chooses.

These are not new rules. The working agreements and `references/code-review-checklist.md` already
imply every one of them, and an agent reading those will reach the same verdict unprompted. What
this list adds is a **gate at a fixed point with a defined action**, at the one moment the agent
has an incentive to skip it — after it has already committed to the fast path.

## Process

### 1. Pre-flight
Identical to `/fledge:fledge` step 1.5, and not negotiable because the ticket is small:

- **Dedicated worktree.** Never work in a checkout another session may be using. Determine the
  default branch with `git symbolic-ref refs/remotes/origin/HEAD`, `git fetch origin <default>`,
  then `git worktree add <path> -b <git-user>/<TICKET>/<slug> origin/<default>`. If the current
  directory is already a dedicated worktree for this ticket's branch, stay put. Branch off the
  default branch unless the user explicitly wants to stack on an in-flight branch.
- **Record the base commit.** `mkdir -p .fledge/phases/<ticket>` and write the branch's fork
  point to `.fledge/phases/<ticket>/.base-commit`:
  `git rev-parse origin/<default> > .fledge/phases/<ticket>/.base-commit`. In the full pipeline
  `/fledge:fledge-implement` does this; lite does not run it, so lite must do it here.
  `references/review-package-format.md` and `references/comment-pass.md` both read this path, and
  the fork point must be pinned before `origin/<default>` moves under you.
- **No stale state.** `.fledge/phases/<ticket>/` (beyond the `.base-commit` just written) and
  `.context/` must be absent or empty for this ticket. If they exist from an earlier run, ask
  before reusing or clearing.
- **GPG.** `git config commit.gpgsign` is `true` *and* `git config user.signingkey` resolves to a
  key in `gpg --list-secret-keys`. Misconfigured → STOP and ask the user to fix it. Never
  `--no-gpg-sign`, never change the setting yourself.

**Skip `/fledge:fledge-auth`.** Lite touches one connector, and step 2 either fetches the source or
fails — a successful fetch proves the connection better than a probe does. A *failed* fetch is when
you run `/fledge:fledge-auth`, to find out why.

### 2. Fetch the source of truth
Fetch it directly with the matching tool (`mcp__linear-server__get_issue`,
`mcp__notion__notion-fetch`, `gh issue view`). **No `SOURCES.md`** — one source needs no manifest,
and there is no source-of-truth question to resolve.

Pull any **linked evidence that pins the defect**: a Sentry event, a failing CI run, a screenshot,
the comment naming the file. Follow it to the exact failing site and confirm it against the code
before planning. This is where lite earns its speed — a stack trace that names the function
replaces the planner subagent's reconstruction of it.

Read the comments, not just the description. Note the SoT's last-updated date; if it predates and
disagrees with the ticket body, that is trip-wire 1.

### 3. Check the trip-wires, then write the plan yourself
Run `## Escalation trip-wires` **first**. Then write `.fledge/phases/<ticket>/PLAN.md` yourself —
**no planner subagent.** Keep the full template's directory layout so an escalation can pick it up
as a seed, but only these sections:

```markdown
# <TICKET>: <title>
## Surface              — frontend | backend | full-stack | library-internal (qa-by-surface.md)
## Source contract      — the SoT ref, its last-updated date, and the requirement in one sentence
## Changes              — bullets, each naming the file path it touches
## Tests to write       — one line per test, naming the regression it catches
## Out of scope         — what this ticket is NOT doing, and where it goes instead
```

`## Surface` is one word and it earns its place twice: it picks the test kind at step 4, and it is
what `/fledge:fledge-qa` or an escalated `/fledge:fledge` reads to route. A second value here (both
frontend and backend) is trip-wire 4.

**No plan review. No plan checkpoint.** The plan is four bullets you wrote from a stack trace; a
three-persona review of it finds nothing, and the user approving it approves nothing they will not
see again at step 9.

If writing the plan makes it grow past the trip-wires, that is the check firing — go back.

### 4. Write the failing tests
Follow `/fledge:fledge-test`'s anti-pattern table and its rationalizations — they apply unchanged.
Lite adds two requirements:

- **Every test names the regression it catches**, in the test name or a one-line docstring.
  `test_removing_a_channel_clears_it_from_the_applied_filter`, not `test_filter_update`. Name it
  *behaviourally*: where the project's own comment rules forbid pointing at a ticket or a plan,
  that applies to a test title too, so the title carries the behaviour and the ticket id stays in
  the commit message.
- **Sort the tests into two kinds, and hold them to different bars.** They are not all red, and a
  skill that demands they are will cost you the ones that matter second-most:
  - **Defect tests** encode the bug the ticket is about. **Run them and confirm each fails for the
    *right* reason** — the real defect, not a missing import, not a fixture error, not a typo in
    the test. An `ImportError` is not red; it is a broken test file. Fix it and re-run before
    writing any production code. At least one test must be of this kind, or you have not
    reproduced the defect and the plan is wrong.
  - **Guard tests** pin behaviour the fix must *not* break — the "and X still works" half of most
    acceptance criteria. These are **green from the moment they are written**, by construction,
    and that is correct, not a defect in them. Do not drop them to satisfy a red bar, and never
    contrive them red. **Step 7 is what gives them teeth**: a guard test nothing can mutate into
    failing is tautological, and step 7 is where that is found out.

Two failed attempts to get a *defect* test failing for the right reason is trip-wire 9. The usual
cause is that the defect is not where the evidence said it was, which means the plan is wrong.

### 5. Implement inline
Write the fix yourself — **no implementer subagent** — until the tests are green. Nothing else
changes: project patterns first, YAGNI, no opportunistic scope. Keep going until green; if the fix
needs a shape the plan didn't describe, re-check the trip-wires before continuing.

Run the comment pass from `references/comment-pass.md` on your own diff before step 6, exactly as
the implementer would. It is a deletion-cost fix, so there is no version of "small ticket" that
buys it out.

Then **commit** — signed, conventional, citing the ticket. This commit is not the release of the
work; step 8 amends it after the review and the mutation check, so the branch still carries **one**
commit at the end. It exists because step 6's package is compiled from `BASE..HEAD`, and a change
that lives only in the working tree compiles to an empty range: `git log` silent, `git diff`
silent, no error anywhere. Review something uncommitted and you review nothing.

### 6. One review round
Compile `REVIEW-PACKAGE.md` per `references/review-package-format.md` — it is a fixed git recipe,
no subagent, and it gives the reviewer a deterministic diff instead of your description of one.

**Check the package is not empty before you spawn anything.** `git log --oneline "$BASE".."$HEAD"`
must name the step-5 commit and `git status --short` must be clean. If the range is empty, stop:
you are about to hand your only reviewer a no-op, and it will come back having found nothing —
which reads exactly like a clean change. The git commands succeed and print nothing, so nothing
else in the run will tell you.

Spawn **one** `fledge-reviewer-adversarial`. Its prompt must carry:

- `mode: code`, artifact path `<phase-dir>/REVIEW-PACKAGE.md`, the project `CLAUDE.md` path,
  output path `<phase-dir>/REVIEW-CODE-final.md`, template `references/templates/review.md`.
- **"You are the only round."** The persona expects a round-1 constructive review to read and to
  avoid re-raising. Tell it there is none, that nothing has been reviewed before it, and that it
  should raise what it finds rather than looking past a round that did not happen. Without this it
  hunts for a `REVIEW-CODE-1-constructive.md` that does not exist.
- **The SoT reference and the decisions already made** — the ticket ref and the `## Changes` and
  `## Out of scope` sections of PLAN.md, so it does not re-litigate settled scope. Pass paths, not
  pasted bodies.
- **The hunt list**, since no integrator will catch what it misses: other callers of every changed
  function, edge cases at the boundary the fix moved, tautological tests (C1), comment altitude
  (C10 → `references/comment-pass.md`), and the project's own service rules from `CLAUDE.md`.

**Act on every finding.** Fix, in this run, every finding whose fix is a deletion or a string edit —
severity follows the cost of the fix ("Severity follows the cost of the fix" in
`~/.claude/shared/working-agreements.md`; applied per-item in `references/severity-rubric.md`
§ Carve-outs), and at that cost there is no argument for deferring. Record
each remaining finding in `REVIEW-CODE-final.md` with a reason: the evidence you have instead, the
ticket it is deferred to, or why it does not apply. An unrecorded finding is a dropped one. A
blocking finding needing a redesign is trip-wire 8.

### 7. Mutation check (mandatory)
Break the fix on purpose, 2–3 targeted ways, and confirm a test goes red for **each**:

- remove the guard the fix added;
- invert its condition;
- hardcode a parameter it threads through.

**Copy the files to a backup first and restore from the backup, not `git checkout`** — a checkout
reverts everything in the path to the last commit, and by this step that is the step-5 commit,
which predates every fix you made for the reviewer at step 6. Those fixes are the most valuable
and least recoverable thing in the tree: they were found by the only review this run gets. On the
MSG-6641 run that recorded this step, a checkout here would have discarded 33 insertions and 30
deletions, including both fixes for the reviewer's two major findings.

A mutation that no test catches is a **blocking finding**: it means the test is tautological and
the regression is unguarded. Fix the test and re-run before step 8.

Checklist item **C1** scores the mutation check as a demand for evidence rather than a blocking
finding, on the grounds that running one costs a real run. In lite that run has already happened by
this step, so the cost argument is spent and blocking is what the severity-follows-cost rule gives
— and with one review round and no integrator, nobody downstream will demand the evidence. Same
rule, same reasoning, different arithmetic. **If you change this, change C1 too** — they are the
same standard under different arithmetic, and the twin-docs rule applies. Record the mutations you ran and what went red; that record
is what the reviewer would otherwise have asked for.

### 8. Verify and commit
On every touched service: lint, format and typecheck (per the project's `CLAUDE.md` and the user's
working agreements — e.g. `ruff check` + `ruff format --check`, `pnpm exec prettier --check`,
`NODE_OPTIONS="--max_old_space_size=8192" pnpm typecheck`). Run the affected tests, not just the
ones you wrote.

Then fold the review fixes and anything step 7 changed into the step-5 commit —
`git commit -S --amend` — so the branch ends with one **signed** commit, conventional message,
citing the ticket. If signing fails, stop — do not work around it. (`REVIEW-PACKAGE.md` keeps the
pre-amend head sha. That is correct: it records what was actually reviewed.)

### 9. 🛑 Checkpoint — "Ship it?"
The only checkpoint. Apply `references/checkpoint-protocol.md` and write
`.fledge/checkpoints/<timestamp>-lite-done.md`. Recap:

- the ticket and the one-sentence requirement;
- the change bullets and the files touched;
- tests added, and the mutations that proved them;
- review findings: fixed vs. recorded-with-reason;
- anything not verified.

Then `AskUserQuestion`, where the question is **how to finish**, not whether the work is right:

- `pr` — open the draft PR now, writing the description with the `dx:pr-description` skill if
  it is installed, otherwise inline from PLAN.md plus the step-10 report;
- `qa` — run browser/API QA first, then come back to this question;
- `iterate` — take feedback and route it back to the right step;
- `abort` — leave the branch as it is.

Draft PRs only, and never ping anyone else.

### 10. Closing report
Print a report in this fixed shape, so lite runs stay comparable to each other and to a full run:

```markdown
## fledge-lite run report — <TICKET>

**Kept:** pre-flight (worktree / branch / GPG) · direct SoT fetch + linked evidence ·
failing-tests-first · comment pass · adversarial review (1 round) · mutation check ·
lint/format/typecheck · signed commit · ship checkpoint

**Skipped, and what it would have caught:**
| Stage | Why it was safe to skip here |
|---|---|
| `/fledge:fledge-auth` | one connector; the step-2 fetch proved it |
| `/fledge:fledge-ingest` + `SOURCES.md` | one source, no SoT question |
| prior-work search | <names the search you did NOT do — see below> |
| `fledge-planner` | <N> change bullets, written from <the evidence that pinned the defect> |
| plan review + plan checkpoint | plan is <N> bullets; the ship checkpoint shows the same scope |
| `fledge-implementer` | <N> lines, one surface |
| constructive + integrator review rounds | one reviewer, findings all actioned or recorded |
| `/fledge:fledge-qa` | <offered at the checkpoint / run / justified skip per qa-by-surface.md> |

**Trip-wires checked:** at step 3 and <when else>. None tripped. | Tripped: <which>.
**Spawns:** 1. **Checkpoints:** 1.
```

Be honest in the skipped column. The **prior-work search is the one drop with real teeth** — it
exists because an earlier design went unfound for a month, and "it looked small" is precisely the
condition under which it gets skipped. Name it every run; if the ticket is in territory anyone may
have investigated before, do the search anyway and move it to the kept list.

## Error handling

- **Source fetch fails** → run `/fledge:fledge-auth` to find out why; fix the connection, don't
  work from the ticket id and memory.
- **The evidence doesn't match the code** (the stack trace's line isn't the failing site) → the
  defect is not where you think. Re-read the source and the comments before planning; if it still
  doesn't resolve, that is more reconstruction than lite is for — escalate.
- **Tests won't fail for the right reason after 2 attempts** → trip-wire 9. Stop and ask.
- **The reviewer returns nothing** → check the package first: `git log --oneline BASE..HEAD`.
  An empty range is the likeliest cause and the silent one — the briefing can be perfect and the
  reviewer still has nothing to read. Then check you briefed it as the only round. A one-round
  review that found nothing on a real fix is more likely under-fed than clean.
- **A trip-wire fires mid-implementation** → stop where you are, commit nothing further, and ask.
  Leave the branch, `PLAN.md` and the tests in place as the seed, and say in the handoff which of
  them are green.

## What this skill does NOT do

- **Replace `/fledge:fledge`.** It is the small-ticket path *into* the same pipeline, and it hands
  back the moment a trip-wire fires. When in doubt about which to run, run full fledge — the cost
  of over-spending on a small ticket is some subagents; the cost of under-spending on a large one
  is a defect in the send path.
- **Ingest, or manage multiple sources.** One source, fetched directly. Two → `/fledge:fledge`.
- **Plan sub-phases, or produce a landing plan.** Anything needing more than one PR is out of
  scope by construction — see trip-wires 2 and 4.
- **Review the plan**, or checkpoint before code. One review round, on the code, at the end.
- **Run QA.** Step 9 offers it; `/fledge:fledge-qa` runs it.
- **Open the PR, or ping anyone.** The user chooses at step 9; draft only, and no one else is
  pinged before the user has reviewed.
- **Lower a standard.** Everything it drops is a *stage*, named in step 10. The checklist, the
  comment pass and the working agreements bind a lite run exactly as they bind a full one.

## Tools needed
- `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`
- The SoT fetcher for the one source (`mcp__linear-server__get_issue`,
  `mcp__notion__notion-fetch`, or `gh`)
- `Agent` (exactly one — `fledge-reviewer-adversarial`)
- `AskUserQuestion` (the single checkpoint, and every escalation)
- `Skill` (`dx:pr-description` at step 9 when installed; `/fledge:fledge-auth` only on a failed
  fetch)

## Related
- Escalates to: `/fledge:fledge --from=plan`
- Borrows unchanged from: `/fledge:fledge-test` (anti-patterns), `/fledge:fledge-review` (reviewer
  input contract), `/fledge:fledge` step 1.5 (pre-flight)
- References: `code-review-checklist.md` (C1, C10, P10), `comment-pass.md`,
  `review-package-format.md`, `checkpoint-protocol.md`, `severity-rubric.md`
- Twin standards live in `~/.claude/shared/working-agreements.md` and
  `references/code-review-checklist.md`. This skill points at them and copies neither.
