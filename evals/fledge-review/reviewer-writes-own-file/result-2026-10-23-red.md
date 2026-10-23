# Eval result: fledge-review / reviewer-writes-own-file — 2026-10-23 (RED)

- **Scenario:** `evals/fledge-review/reviewer-writes-own-file/scenario.md`
- **Mode:** mechanical plus transcript audit, single arm. See the scenario for why this is not an A/B.
- **Version under test:** 0.9.1 (`agents/` unchanged on this point since `e0259f9`, the initial commit)
- **Transcripts audited:** every `fledge:fledge-reviewer-*` subagent under
  `~/.claude/projects/*/*/subagents/` on 2026-10-23: 239 runs across about 25 sessions. A few
  sessions appear under two project directories after a worktree move, so counts are approximate
  in the last digit. The spread is what matters.

## Scorecard

| Pass criterion | 0.9.1 |
|---|---|
| 1. Every persona given an output path has `Write` | **fail**: constructive and adversarial do not; only the integrator does |
| 2. Each persona writes its review with `Write` | **fail**: 0 of 163 constructive and adversarial runs; 141 used a `Bash` heredoc |
| 3. No persona hands the review back as text instead | **fail**: 21 runs did, in 11 sessions |
| 4. The file exists when the round returns | **partial**: yes after a heredoc; after a text hand-back, only if the orchestrator chose to save it |

0 of 4.

## Criterion 1: the allowlists

```
agents/fledge-reviewer-constructive.md  tools: Read, Bash, Glob, Grep, WebFetch, mcp__notion__…, mcp__linear-server__…
agents/fledge-reviewer-adversarial.md   tools: Read, Bash, Glob, Grep, WebFetch, mcp__notion__…, mcp__linear-server__…
agents/fledge-reviewer-integrator.md    tools: Read, Write, Bash, Glob, Grep, WebFetch, mcp__notion__…, mcp__linear-server__…
```

| Skill step | Output path | Persona | `Write`? |
|---|---|---|---|
| `fledge-review` step 3 | `REVIEW-<MODE>-1-constructive.md` | constructive | no |
| `fledge-review` step 4 | `REVIEW-<MODE>-2-adversarial.md` | adversarial | no |
| `fledge-review` step 5 | `REVIEW-<MODE>-final.md` | integrator | yes |
| `fledge-lite` step 6 | `REVIEW-CODE-final.md` | adversarial | no |

Neither `git log` nor `CHANGELOG.md` gives a reason for the difference. The `tools:` line has not
changed since the initial commit.

## Criteria 2–4: what the runs did

| Persona | `Bash` heredoc | `Write` | Returned text only |
|---|---|---|---|
| constructive | 65 | 0 (1 attempt, refused) | 7 |
| adversarial | 77 | 0 | 14 |
| integrator | 7 | 68 | 2 |

The one constructive round that reached for `Write` first (`agent-ae3a52d4aafae1afc`, writing
`REVIEW-PLAN-1-constructive.md`) got:

```
<tool_use_error>Error: No such tool available: Write. …</tool_use_error>
```

and fell back to `Bash`.

**The heredoc path** is what MSG-6641 did, the 0.9.0 lite release-verification run. Its single
adversarial round (`agent-a79a89c672355974d`, session `5aa4badd`) wrote the lite final review
like this:

```
Bash: cat > …/msg-6641-android-notification/.fledge/phases/MSG-6641/REVIEW-CODE-final.md <<'MDEOF'
# Adversarial review — 2026-10-19T15:02:00Z
```

The file existed, so the run worked. It worked by routing around the allowlist, and since `Bash`
is unrestricted, leaving out `Write` restricted nothing.

**The text hand-back path** is the defect. In session `53a5719f` (MSG-6523), the constructive
round reasoned from the missing tool to a prohibition:

> I did not write `REVIEW-CODE-1-constructive.md`, because this harness bars writing report
> files. The full review is below. The orchestrator can save it to …

The orchestrator saved it, but only because it had already learned to expect this and patched
its own spawn prompt:

> Output: …/REVIEW-CODE-1-constructive.md. If your harness forbids writing it, return the full
> review text in your final message.

Neither skill says that. It is option (b) improvised in one session's prompt, and the next
session's orchestrator starts without it. In a run without that patch, the integrator is told to
read a round-1 file that nobody wrote.

## Verdict

RED, 0 of 4. The contract "write your review to this path" names a tool two of the three personas
do not have. Most runs cover the gap with a heredoc; about one in eight hands the review back as
text and depends on the orchestrator to notice. Fix: give both personas `Write`, as the integrator
already has, and have each skill check the file exists before the next round reads it.
