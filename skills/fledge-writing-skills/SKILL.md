---
name: fledge-writing-skills
description: Use when the user says "write a skill", "add a fledge skill", "improve <skill>", "the agent keeps doing X — fix the skill", or after an eval surfaces a behavioral gap. Authors or revises a fledge skill, agent, or reference via a RED→GREEN→REFACTOR loop (document the baseline failure first), encoding the fledge conventions and the trigger-first description rule.
---

# fledge-writing-skills

A skill is **process documentation that changes how an agent behaves**. Authoring one is TDD applied to prose: you do not write a skill until you have watched the agent fail without it, and you do not call a skill done until you have closed the loopholes it invents.

The cardinal rule: **no skill (or skill change) without a documented baseline failure first.** A skill written from imagination — "the agent might do X wrong" — encodes a guess, not a fix. Run the scenario, watch the real failure, fix exactly that.

## Preconditions

- You are editing inside the fledge plugin repo (skills live in `skills/<name>/SKILL.md`, personas in `agents/<name>.md`, shared rules in `references/`).
- You can spawn subagents (the `Agent` tool) to observe baseline behavior — this is how RED is measured. See `/fledge:fledge-eval` and `references/skill-eval-protocol.md`.

## The authoring loop: RED → GREEN → REFACTOR

### RED — document the baseline failure
Run the target scenario *without* the skill (or with the current version, if you're revising) and capture how the agent naturally fails. Use `/fledge:fledge-eval` to drive this with a fresh subagent, or do it by hand and record the transcript excerpt.

- Write the scenario down as an eval fixture under `evals/<skill>/` (see `references/skill-eval-protocol.md`) **before** you touch the skill.
- The failure must be concrete and observable: "the agent wrote a test that calls a private `_helper()` directly" — not "the agent might write bad tests."
- If you cannot produce a failure, stop. There is nothing to fix, and a skill that fixes nothing is pure context cost.

### GREEN — write the minimal skill that fixes exactly those failures
- Add only the instruction needed to flip the documented failure to a pass. Resist writing the skill you imagine you'll want.
- Re-run the scenario *with* the skill. Confirm the behavior changed. That delta is the evidence the skill works — record it in the eval result.
- If behavior didn't change, the instruction is too weak or buried. Strengthen or move it; don't pile on more text.

### REFACTOR — close the loopholes
Skills leak. Once an instruction exists, the agent invents rationalizations to route around it ("this private function is *basically* public", "the failure mode doesn't apply here"). Re-run and watch for them.

- For each new rationalization, add a counter — usually a one-line "Rationalizations to reject" entry, not a paragraph.
- Stop when re-runs stop surfacing new evasions, not when the doc feels thorough.

## Fledge conventions (match these exactly)

### SKILL.md
- **Location:** `skills/<name>/SKILL.md`. Skill name = directory name = frontmatter `name`.
- **Frontmatter:** `name` and `description` only. **No `version:` field** — the plugin version in `.claude-plugin/plugin.json` is the single release unit.
- **The description leads with a "Use when …" clause.** Triggers first (verbatim user phrases, the upstream skill that precedes it, error conditions), then a short clause disambiguating what the skill does. The model routes on the description *before* it reads the body, so the triggers must come first — vague or buried triggers mean the skill won't fire when it should, or fires when it shouldn't. Every fledge skill follows this; read the existing descriptions before writing a new one.
- **Body structure** (omit sections that don't apply, keep the order):
  `# <name>` → one-paragraph framing → `## Preconditions` → `## Arguments` → `## Process` (numbered steps) → `## Error handling` → `## What this skill does NOT do` → `## Tools needed` → `## Related`.
- **Voice:** terse, declarative, opinionated. Imperative mood ("Read the plan", not "You should read the plan"). State the rule, then the rationale in a clause — not a preamble.
- `## What this skill does NOT do` is load-bearing: it draws the boundary against neighboring skills and prevents scope creep. Always include it for a pipeline skill.

### Agent personas
- **Location:** `agents/<name>.md`. Frontmatter: `name`, `description`, and an explicit `tools:` allowlist (least privilege — only the tools the persona needs).
- Body opens with the persona framing (`# <name> — <role> persona`), then `## Your job`, `## Inputs (passed in your prompt)`, `## Process`. The persona is the *character and judgment*; the spawning skill passes the runtime context.
- A persona receives **references, not pasted content** — pass file paths and let the agent fetch/read. (Mirrors the context-budget protocol.)

### Reference docs
- **Location:** `references/<topic>.md`. A reference is the **single source of truth** for one cross-cutting rule that multiple skills/agents share.
- State at the top which skills/agents defer to it (see `references/qa-by-surface.md` for the model).
- Create a reference only when ≥2 consumers share the rule. One consumer → keep it inline in that skill (YAGNI).

## Releasing a skill change

Every content change merged to `main` must (the installed plugin is a version-gated cache — see `README.md` § Releasing changes):

1. Bump `version` in **both** `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` (keep them in sync). Minor bump for a new skill/surface, patch for a fix.
2. Add a `CHANGELOG.md` entry under a new version heading.
3. If you added a skill, agent, or reference, add it to the relevant `README.md` table/list.

## Rationalizations to reject

| Rationalization | Reality |
|---|---|
| "I can see the failure mode, I don't need to actually run it." | Imagined failures encode guesses. Run the scenario; fix what actually happens. RED is non-optional. |
| "I'll write the full skill now and trim later." | Every unused instruction is permanent context cost on every invocation. Write the minimal fix; add more only when a new failure demands it. |
| "The skill reads well, so it works." | Reading well ≠ changing behavior. The only proof is a with/without delta. Re-run and check. |
| "This rule is general, it deserves its own reference doc." | One consumer doesn't justify a shared reference. Inline it until a second consumer appears. |
| "I'll add a `version:` to the skill so it's tracked." | Skills carry no version frontmatter. The plugin version is the single release unit. |
| "I'll open the description with what the skill does." | Descriptions lead with the `Use when …` triggers — that's what the model routes on before it reads the body. Triggers first, disambiguate after. |

## What this skill does NOT do
- Run the eval itself — that's `/fledge:fledge-eval` (this skill tells you *how to author*; the eval skill *measures the behavior*).
- Bump the plugin version or write the CHANGELOG for you — it tells you to; do it.
- Author project source code or pipeline artifacts — it authors fledge's own skills/agents/references.

## Tools needed
- `Read`, `Write`, `Edit`, `Glob`, `Grep` (author the files)
- `Agent` (observe baseline behavior for RED, confirm the delta for GREEN)

## Related
- Harness: `/fledge:fledge-eval`
- References: `skill-eval-protocol.md`
- Release mechanics: `README.md` § Releasing changes
