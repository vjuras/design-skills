# design-skills

A pipeline of portable Agent Skills that take a feature from idea to merged code, keeping the **design doc as the source of truth** at every step. Each skill does one job, hands off cleanly to the next, and never silently makes decisions the human owns.

The skills are code-first (every claim traces to a `file:line`), decision-driven (forks are surfaced as explicit options for the human to pick), and verification-gated (`[x]` means "verified against spec," not "looks done").

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| [`/design-doc`](./design-doc) | AI | analyze codebase; surface and draft decision options |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| [`/design-cleanup`](./design-cleanup) | AI | collapse working draft to slim final form — optional |
| [`/design-task`](./design-task) | AI | translate decisions into phased, dependency-ordered impl tracker |
| [`/write-code`](./write-code) | AI | implement following on-doc execution rules |
| [`/design-task-check`](./design-task-check) | AI | verify each phase against spec; tick `[x]` on match, `[~]` on drift |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |
| [`/design-work-loop`](./design-work-loop) | AI | loop `/write-code` + `/design-task-check` per phase until all phases pass — experimental |

## Core principles

These run through every skill in the pipeline:

1. **Code first, prose second.** Every claim traces to a `file:line` or symbol, or carries an explicit `<assumption>` flag.
2. **Decisions are human-owned.** Skills recommend; the human decides. No skill flips `— PROPOSED` to `— RESOLVED` without an explicit pick.
3. **The doc is the deliberation surface.** Options, pros/cons, risks, and recommendations live in the doc — not in narrow chat-side multiple-choice prompts.
4. **Process-shaped, not essay-shaped.** Tables, checkboxes, and decision gates over prose. One paragraph per rationale, one sentence per bullet.
5. **`[x]` is earned, not asserted.** Only `design-task-check` ticks boxes, and only against verified spec match.
6. **Each skill does one job and hands off.** No skill smuggles another's responsibility (authoring vs. cleanup vs. execution vs. verification).

## How it works across tools

Every skill follows the Agent Skills standard — plain `name` + `description` frontmatter, a markdown body, and progressive disclosure. The same `SKILL.md` works in Claude Code (`~/.claude/skills/`), Cursor (`~/.cursor/skills/`), and any other agent that respects the spec. Auto-invocation is driven by the frontmatter `description`; the body loads on demand when the agent matches the user's intent against it. No vendor-specific frontmatter fields are used, so the skills are portable as-is.

## Install

Clone the repo and symlink (or copy) the skill folders into your agent's skills directory:

```bash
git clone <repo-url> design-skills

# Claude Code
ln -s "$(pwd)/design-skills"/*/ ~/.claude/skills/

# Cursor
ln -s "$(pwd)/design-skills"/*/ ~/.cursor/skills/
```

Each skill is self-contained in its own folder (`SKILL.md` + optional `template.md` + `README.md`), so you can also install just the ones you want.

## Layout

```
design-skills/
├── design-doc/          # author the design doc (template.md + SKILL.md)
├── design-cleanup/      # collapse working draft → final form
├── design-task/         # design doc → phased impl tracker (template.md)
├── write-code/          # coding-hygiene reminders
├── design-task-check/   # verify tracker items against code
└── design-work-loop/    # experimental phase loop orchestrator
```

Each folder's `README.md` documents that skill in detail. `SKILL.md` holds the full instructions the agent follows; `template.md` (where present) is a pure markdown shape reference.

## License

[MIT](./LICENSE) — use, modify, and distribute freely.
