# design-work-loop

> **Experimental.** Wraps `/write-code` and `/design-task-check` in a per-phase loop until every checklist item in an impl doc is `[x]` — or a stop condition halts the loop.

Sequences the implement-then-verify cycle phase-by-phase. Does **not** write code itself and does **not** tick boxes itself — only orchestrates the two sub-skills that do, with hard stop conditions on drift, missing info, design forks, and blockers.

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| `/design-doc` | AI | analyze codebase; surface and draft decision options |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| `/design-cleanup` | AI | collapse working draft to slim final form — optional |
| `/design-task` | AI | translate decisions into phased, dependency-ordered impl tracker |
| `/write-code` | AI | implement following on-doc execution rules |
| `/design-task-check` | AI | verify each phase against spec; tick `[x]` on match, `[~]` on drift |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |
|------|-------|-------------|
| **`/design-work-loop`** | **AI** | **loop `/write-code` + `/design-task-check` per phase until all phases pass — experimental** |
|------|-------|-------------|

> **Status: experimental.** This skill replaces the default human-in-the-loop review between phases with hard stop conditions (drift, missing info, design fork, blocker, smuggled scope). It is **not** a replacement for `/write-code` and `/design-task-check` — it invokes them. Use when the impl doc is mature and per-phase review wouldn't change anything; drive the two sub-skills manually when you still want phase-level oversight. Confirm before invoking — `/write-code` edits the working tree and there is no "undo" baked into the loop. Known sharp edges: long-running loops can grow context fast; `/write-code`'s stop-and-ask events surface but block progress (intended); drift halts immediately and does not retry.

## What this skill does

- Reads the impl doc; refuses to run unless every source design doc is `RESOLVED`.
- Builds a phase queue from `### Phase N` headings; skips phases where every item is already `[x]`.
- For each phase: invokes `/write-code` (scoped to that phase's items), then `/design-task-check`. Advances only when the check ticks every current-phase item to `[x]` with no drift.
- Halts loudly on: drift (`[~]`), `/write-code` stop-and-ask (missing info), design fork, unexpected complexity, smuggled scope, or check-itself stop. Never auto-fixes, never auto-answers.
- At the end, runs one final full-doc check pass and proposes the status header flip — but never flips without explicit user "yes."
- Hands off to `/create-pr` once everything is verified.

## How it works across tools

Follows the Agent Skills standard — plain `name` + `description` frontmatter, markdown body, progressive disclosure. The same `SKILL.md` works in Claude Code (`~/.claude/skills/`), Cursor (`~/.cursor/skills/`), and any other agent that respects the spec. Auto-invocation is driven by the frontmatter `description`; the body loads on demand when the agent matches the user's intent against it. No Claude-specific frontmatter fields are used, so the skill is portable as-is.

## Files

- `SKILL.md` — full instructions (workflow, stop conditions, anti-rationalizations).
- (No `template.md` — this skill orchestrates, doesn't author.)

## Trigger phrases

`/design-work-loop`, `/work-loop`, "execute the impl doc end-to-end", "loop through the phases until done", "auto-implement the impl doc".
