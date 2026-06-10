# design-cleanup

Collapse a fully-resolved design doc (every `D{n}` is `— RESOLVED → Option <X>`) into the slim final form per `../design-doc/template.md`. Purely mechanical — does not make new decisions, re-analyze code, or rewrite prose for style.

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| `/design-doc` | AI | analyze codebase; surface and draft decision options |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| **`/design-cleanup`** | **AI** | **collapse working draft to slim final form — optional** |
| `/design-task` | AI | translate decisions into phased, dependency-ordered impl tracker |
| `/write-code` | AI | implement following on-doc execution rules |
| `/design-task-check` | AI | verify each phase against spec; tick `[x]` on match, `[~]` on drift |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |

## What this skill does

- Validates input contract: every `D{n}` is `— RESOLVED`, no working-draft scratch left unanswered.
- Mechanical collapse: working-draft `D{n}` block → resolved-shape per `../design-doc/template.md §3`. Rejected options' bodies become one-line "Why not" rows in the `Options evaluated` table.
- Removes scratch sections (`## Open questions`, `## Pending decisions`).
- Picks optional sections (Inventory / Mapping / Validation / etc.) from the selection table by trigger.
- Flips status header to `RESOLVED` (or `RESOLVED — applied to <symbol/file>`).
- Stops with a routing message if any decision is still `— PROPOSED`.

## Files

- `SKILL.md` — full instructions.
- (No own `template.md` — uses `../design-doc/template.md` as the target shape.)

## Trigger phrases

`/design-cleanup`, "finalize this design doc", "collapse to final form".

Also auto-routes here from `/design-doc` when all `D{n}` are resolved and scratch is still present.
