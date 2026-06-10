# design-doc

Author a feature design document by analyzing existing code first, surfacing every decision point that the code cannot answer alone, and drafting each as options with pros/cons/risks (one marked as recommendation) — directly in the doc. The detailed working draft is the primary deliverable; collapsing to slim final form is opt-in via `/design-cleanup`.

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| **`/design-doc`** | **AI** | **analyze codebase; surface and draft decision options** |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| `/design-cleanup` | AI | collapse working draft to slim final form — optional |
| `/design-task` | AI | translate decisions into phased, dependency-ordered impl tracker |
| `/write-code` | AI | implement following on-doc execution rules |
| `/design-task-check` | AI | verify each phase against spec; tick `[x]` on match, `[~]` on drift |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |

## What this skill does

- Frames the doc (subject, scope, location, sibling-doc tone reference).
- Explores the codebase via subagents; logs `## Open questions` and `## Pending decisions`.
- Drafts every pending decision as a `D{n}` block with options, pros/cons/risks, and a recommendation.
- Hands off to the user — **never** flips `— PROPOSED` to `— RESOLVED` without an explicit user pick.
- Re-invocation safe: Phase 0 detects doc state and routes (stub → frame; partial → resume; fully resolved → tells user to run `/design-cleanup`).

## Files

- `SKILL.md` — full instructions (workflow, principles, anti-rationalizations).
- `template.md` — pure markdown skeleton (universal layout + optional section shapes).

## Trigger phrases

`/design-doc`, "write a design doc for X", "design <feature>", "document the decisions for Z".
