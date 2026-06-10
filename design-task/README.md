# design-task

Prepare an implementation tracker doc from a resolved design doc. Translates resolved decisions into a phased, dependency-ordered checklist with concrete tasks, "what changes where" manifest, and shape-fitting optional sections (contracts, mapping, validation, tests, security pass). **This skill prepares the plan; it does not execute.**

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| `/design-doc` | AI | analyze codebase; surface and draft decision options |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| `/design-cleanup` | AI | collapse working draft to slim final form — optional |
| **`/design-task`** | **AI** | **translate decisions into phased, dependency-ordered impl tracker** |
| `/write-code` | AI | implement following on-doc execution rules |
| `/design-task-check` | AI | verify each phase against spec; tick `[x]` on match, `[~]` on drift |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |

## What this skill does

- Loads upstream design doc(s); refuses to run if any `D{n}` is unresolved.
- Maps current codebase state via subagents (one per area: contracts, mapping, validation, persistence, tests).
- Picks a phase shape from the menu (Backend feature / Frontend / Domain refactor / Infra migration). Adapts freely; phase counts are not fixed.
- Drafts the impl tracker doc with the on-doc execution rules baked in (so the executing agent doesn't need any design-* skill loaded — it follows the rules in the doc itself).
- Gates on a scope-discipline filter: every checklist item must trace to a `D{n}` and name a concrete file/symbol/area.
- Hands off; refuses to start coding even if asked.

## Files

- `SKILL.md` — full instructions.
- `template.md` — pure markdown skeleton (universal skeleton including the on-doc `## How to execute this doc` rules + optional section shapes).

## Trigger phrases

`/design-task`, "prepare implementation plan for X", "draft the task breakdown for Y", "build a tracker doc from the design".
