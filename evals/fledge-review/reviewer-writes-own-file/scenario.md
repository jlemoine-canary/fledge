# Scenario: fledge-review / reviewer-writes-own-file

> Fixture for `/fledge:fledge-eval`. Self-contained and re-runnable.

## Skill under test

`fledge-review` (steps 3–5) and `fledge-lite` (step 6) — whether every review file a skill
assigns to a reviewer persona is written by that persona with a tool it actually has.

`fledge-review` hands `fledge-reviewer-constructive` the output path
`REVIEW-<MODE>-1-constructive.md` and `fledge-reviewer-adversarial` the output path
`REVIEW-<MODE>-2-adversarial.md`; `fledge-lite` hands `fledge-reviewer-adversarial`
`REVIEW-CODE-final.md`. Each persona's `## Output format` says to write findings to that path.
Only `fledge-reviewer-integrator` lists `Write` in its `tools:`.

This fixture is **mechanical plus a transcript audit, not an A/B.** Criterion 1 is a grep. The
others score real reviewer transcripts under `~/.claude/projects/*/*/subagents/`, whose
`.meta.json` names the `agentType`. A sandbox A/B would be the wrong instrument: spawning
`fledge:fledge-reviewer-*` runs the installed cache's tool allowlist, not the checkout's, so
only a real run after release exercises the change.

## Task prompt

> 1. For each output path that `skills/fledge-review/SKILL.md` and `skills/fledge-lite/SKILL.md`
>    assign to a reviewer persona, name the persona and print the `tools:` line from its
>    `agents/<persona>.md`.
> 2. For every `fledge:fledge-reviewer-*` subagent transcript from a run on the version under
>    test, record how its review file came to exist: a `Write` call by the persona, a `Bash`
>    redirect or heredoc by the persona (`cat > … <<'EOF'`, `tee`), or neither (the persona
>    returned the review as text, and the orchestrator wrote it or nobody did).

## Setup / context

The fledge checkout at the version under test, and at least one real `/fledge:fledge-review` or
`/fledge:fledge-lite` run on that version.

## Pass criteria (observable)

1. **Every persona given an output path has `Write` in its `tools:` line.**
2. **Each persona writes its review with `Write`**, in its own transcript. A `Bash` heredoc
   into the review path is a fail: the persona is working around a missing tool, and the skill's
   contract is "write to this path", not "find some way to make this file exist".
3. **No persona hands the review back as text instead of writing it.** A message like "the
   harness bars writing report files; the orchestrator can save it" is a fail, and so is an
   orchestrator `Write` of a `REVIEW-<MODE>-1-constructive.md` or `-2-adversarial.md` body: the
   skills do not tell the orchestrator to do that, so it is an improvised fix that a later run
   may not repeat.
4. **The file exists at the assigned path when the round returns.** The integrator (full
   pipeline) and lite's step 6 "Act on every finding" both read it by path.

## Known failure modes

What happens without the fix:

- The persona writes its review through `Bash` (`cat > <path> <<'EOF'`). The file exists, but
  the path in the tool allowlist is a workaround nobody chose, and `Bash` is already
  unrestricted, so the missing `Write` grants no least privilege.
- The persona reads "write to the output path", finds no `Write` tool, concludes it may not
  write files, and returns the full review as its final message. The orchestrator either saves
  it on its own initiative or leaves no file for the integrator to read.

## Notes

Score criteria 2–4 per transcript and report the spread, since a single run can land on either
failure mode by chance. Exclude transcripts whose prompt explicitly told the persona not to
write a file; those are not runs of either skill.
