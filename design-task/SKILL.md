---
name: design-task
description: Prepare an implementation tracker doc from a resolved design doc. Translates resolved decisions into a phased, dependency-ordered checklist with concrete tasks, "what changes where" manifest, and optional sections (contracts, mapping, validation, tests, security pass) picked to fit the task type. Works for backend, frontend, domain-only refactors, infra migrations, or any task — phase shapes are reasoned per task, not prescribed. The skill PREPARES the doc; execution is delegated to a separate session/agent. Use when the user says "/design-task", "prepare implementation plan for X", "draft the task breakdown for Y", "build a tracker doc from the design".
---

# Design Task — Prepare Implementation Tracker

Take a resolved design doc; produce an implementation tracker doc — phased checklist, "what changes where" manifest, optional shape-fitting sections.

**This skill prepares the plan. It does not execute it.** Building, linting, running tests, opening PRs — out of scope. Execution is a separate session, run by another agent or human, using the produced doc as the source of truth.

`template.md` is a markdown shape reference. This file owns: when to use, what to include, how phases are reasoned, hand-off rules.

## When to use

Trigger phrases: `/design-task`, "prepare implementation plan", "draft the task breakdown", "build a tracker doc from the design", "phase out the implementation".

**When NOT to use:**
- Do not use to actually implement — execution is a separate session.
- Skip when the design doc is still `DRAFT` or has unresolved `D{n}` — route back to `design-doc`.
- Skip for hot-fix or one-line bug fixes that don't warrant phased planning.
- Skip when an existing impl doc already covers the work — invoke `/design-task-check` instead.

## Prerequisite: a resolved design doc

`design-task` consumes the output of `design-doc`. Before starting, confirm:

- A design doc exists with all `D{n}` sections marked `RESOLVED`.
- The user has signed off (status header is `RESOLVED`, not `DRAFT`).
- No `Open questions` of substance are blocking implementation.

If any of these fail, stop and route back to `design-doc`. Don't paper over open questions with assumptions.

## Core principles (do not skip)

1. **Plan, don't execute.** The deliverable is a tracker doc. If the user asks you to start coding, stop and hand off.
2. **Design doc is law.** Every checklist item traces to a `D{n}` decision or a structural consequence of one. Forks the design didn't cover → stop, route back to `design-doc`. The impl doc never out-runs the design.
3. **Phase shape fits the task.** Phase shape is reasoned per task from dependency boundaries. §Phase patterns is examples, not a menu.
4. **The impl doc is process-shaped, not essay-shaped.** Tables, checkboxes, file/symbol references. Every checklist item names a file, symbol, or specific area. *"Implement X"* is not a task; *"Add `<rule>` in `<file>` returning `<code>` for `<field>`"* is. The reader is about to *do* something — not consume an essay.
5. **Optional sections are optional.** Contracts, mapping, validation, tests, security pass, error mapping — included only when the trigger fires. Empty or filler sections are worse than missing ones.

---

## Workflow

### Phase 1 — Load upstream design

- Locate the resolved design doc(s). If multiple, list them and confirm scope with the user (one feature often spans 2–4 design docs).
- Verify status across all `D{n}` is `RESOLVED`. If any is `PROPOSED` or has blocking `Open questions`, stop and route back to `design-doc`.
- Note the doc paths — they go into the `Sources:` block of the impl doc.

### Phase 2 — Map current codebase state

For non-trivial tasks, delegate exploration to parallel sub-tasks / sub-agents if your environment supports them — one per area. Areas vary by task type:

- **Backend-API:** request/response layer, mapping/translation, validation, domain logic, persistence, tests
- **Frontend-UI:** components, state/store, API client, styles/tokens, routing, tests
- **Domain-only refactor:** entities/types, business logic, callsites, tests
- **Infra / migration:** schema/config, write path, read path, observability, runbooks

The output feeds the doc's "current state" cells — every checklist item eventually has a `Live reference` once shipped.

### Phase 3 — Reason the phase shape

Reason phase shape from the design's dependency boundaries: what must exist before what. Decide phase count, names, order. §Phase patterns is examples — don't pick off the shelf. Phase counts are not fixed — small refactors might be 3 phases, big features 8+.

If the design doc clearly implies a task shape (e.g. resolved decisions all point at a backend-API feature, or all at a domain refactor), match an example as a starting point and proceed without prompting. State the chosen shape inline at the top of the impl doc so the user can challenge it on review.

Confirm with the user only when the shape is ambiguous — for example: design touches both a backend surface and an FE consumer, or it's unclear whether the task is "feature" vs "refactor". One short message:

> "Design spans BE contracts and FE form changes — phase as one combined doc (P1 BE contracts → P2 FE wiring → ...) or split into two impl docs?"

### Phase 4 — Build the implementation doc

**Scope-discipline gate.** While building the doc, if you encounter a fork the upstream design didn't resolve — **stop**. Do not invent a "new decision" inside the impl doc. Route back to `design-doc` to drive the decision; resume here when the new `D{n}` is `RESOLVED`. The impl doc never out-runs the design.

For every candidate checklist item, run this filter before adding it:

| # | Question | If no |
|---|---|---|
| 1 | Does this trace to a `D{n}` decision or a structural consequence of one? | Drop it. The impl doc only encodes design-resolved work. |
| 2 | Is the file / symbol / specific area named? | Refine before adding. *"Implement X"* is not a task. |
| 3 | Is it inside the scope of the loaded design doc(s)? | If the user wants adjacent work tackled, draft a separate impl doc or a new design doc — don't smuggle it in. |

Use `template.md`. Required sections:

- **Sources block** — links to design docs.
- **Status header** — `DRAFT — ready for execution`.
- **`## How to execute this doc`** — copy verbatim from `template.md §1`. This is the on-doc contract for any agent (or human) who picks the doc up to execute. Don't paraphrase — the rules are load-bearing and the agent reading them may not have any of the design-* skills loaded.
- **Overview** — 1–3 sentences + scope/area table.
- **Implementation checklist** — phases reasoned in Phase 3, populated with concrete tasks (file/symbol/area named, not "do the thing").
- **Summary "what changes where"** — file-by-file change manifest.

Optional sections — include only when their trigger fires; see selection table below.

Save next to the design doc(s).

### Phase 5 — Verify before handing off

Before announcing the impl doc is ready for execution, run these checks. Evidence required — "looks right" is not sufficient.

1. **Status header is `DRAFT — ready for execution`.** Doc contains exactly one status line reading `> **Status:** DRAFT — ready for execution` (or accepted variant per `template.md §2`).
2. **Sources block links to every consumed design doc.** Sources block exists and lists every design doc consumed. Read each listed path — confirm each is `RESOLVED` (or `RESOLVED — applied to ...`).
3. **`## How to execute this doc` section is present and matches `template.md §1` verbatim.** Doc contains exactly one `## How to execute this doc` section. Re-read the six numbered rules — they are the contract for any executing agent. Do not paraphrase.
4. **Every phase has at least one concrete task.** Read each `### Phase N:` heading. If a phase has no checklist or only placeholder items, the phase shape is wrong — re-think the phase shape or drop the phase.
5. **`## Summary — what changes where` exists and is comprehensive.** Section is present; every checklist item's target file/area appears in it.
6. **Optional sections trigger-checked.** For each optional section present, re-verify its trigger from §Picking optional sections. If a trigger does not fire, drop the section.

If any check fails, fix and re-run.

### Phase 6 — Hand off

When all six checks pass, post a one-line message: doc path + a note that execution happens in a separate session using the impl doc as the source of truth (the on-doc `## How to execute this doc` rules brief the executor and route them through `/design-task-check`).

**This skill ends here. Do not start implementing.** If the user replies *"and now run it"* — stop, point at the doc, suggest a fresh session for execution.

---

## Phase patterns

Examples, not a menu. Reason phasing from the task's dependency boundaries first; cross-check examples after. **Unit tests live inside the phase that builds what's tested** — never a standalone phase by default.

### Backend feature (request/response surface)

| # | Phase | Typical work |
|---|---|---|
| P1 | Contracts | Request/response types, constants, registries; unit tests for type coercion / constant maps |
| P2 | Mapping / translation | external ↔ internal representations; unit tests for mappers |
| P3 | Validation | Input-shape rules + semantic rules; unit tests per rule |
| P4 | Error mapping | Issue/error keys → user-facing codes/messages/field paths; unit tests for mapping |
| P5 | Application logic | Creators / replacers / patchers; new entity calls; unit tests per case |
| P6 | Surface wiring | Routes, handlers, dependency injection; end-to-end tests per surface |
| P7 | Doc + status | Flip status, fill `Live reference` cells |

### Frontend feature

| # | Phase | Typical work |
|---|---|---|
| P1 | Types / API client | Type defs, client methods; unit tests for client methods |
| P2 | State / store | Reducers, hooks, store slices; unit tests for reducers/hooks |
| P3 | Components | Presentational + container components; component tests |
| P4 | Routing / navigation | Routes, deep links |
| P5 | Styling / design tokens | New variants if needed |
| P6 | Doc + status | Flip status |

### Domain-only refactor (no public surface change)

| # | Phase | Typical work |
|---|---|---|
| P1 | New types / interfaces | Entities, value objects |
| P2 | Logic migration | Move/rename functions; behaviour-preserving; existing tests stay green; add tests for new structure |
| P3 | Caller updates | Adapt every callsite to new shape; update affected callsite tests |
| P4 | Cleanup | Remove dead code, deprecation notes |

### Infra / migration / data task

| # | Phase | Typical work |
|---|---|---|
| P1 | Schema / contract change | New table, new field, new config key |
| P2 | Forward-compatibility writes | Dual-write or shadow-write |
| P3 | Backfill | One-shot or rolling job |
| P4 | Read switch | Flip readers to new source |
| P5 | Cleanup | Remove old writes/reads |
| P6 | Observability | Metrics, alerts, runbooks |

---

## Picking optional sections

`template.md` lists optional section shapes. Include each only when its trigger fires.

| Optional section | Include when |
|---|---|
| Contracts (typed structs / signatures) | Task changes a request/response, component prop shape, or public function signature |
| Field mapping table | Task translates between two representations (DTO ↔ entity, API ↔ domain, legacy ↔ new) |
| Layer-assignment table | Task spans multiple validation/processing layers and the split needs documenting |
| Field-error table | Task introduces validation that returns user-facing error codes/messages |
| Error mapping (internal → external) | Task adds new error classes to a system with an internal→external translation layer |
| Mode / variant table | Task has discriminator-driven shape (different fields per mode) |
| Phased rollout | Task ships in distinct rollout phases with different contracts |
| Test plan section | Task touches code that needs tests AND the per-layer required-tests need documenting (not every task needs an explicit table — small features can just list tests in the checklist) |
| **Security pass** | See "When to include security pass" below |
| HISTORICAL | Pre-redesign shape worth preserving for context |
| Cross-doc references | Task depends on or supersedes sibling docs |

**Default posture:** include only the universal sections (Sources, Status, Overview, Implementation checklist, Summary). Add optional sections one at a time as their trigger fires.

### When to include the security pass

Include only when the task does any of:

- Adds or modifies an external surface (HTTP, gRPC, public SDK, FE form posting to a server, CLI command).
- Touches an authz boundary (new permission, new role check, new tenant-isolation rule).
- Handles user-controlled input that flows into queries, file paths, shell commands, HTML/JS rendering, or template strings.
- Mutates audit-relevant state (ownership, billing, permissions, content visibility).
- Stores or returns secrets, tokens, or PII.

**Skip** on:

- Pure internal refactors with no behaviour change.
- Doc / comment / type-only changes.
- Test-only changes.
- Pure visual/styling changes that don't reach the network.

When included, populate from this menu — pick only the items that fit the task:

- **Authz / permission scope** — new mutating surfaces enforce the right scope or role.
- **Ownership / tenancy** — referenced IDs verified against the current tenant before use.
- **Read-only invariants** — server-/system-controlled fields reject writes from outside.
- **Input boundaries** — max-length / regex / enum / schema gates at the entry layer.
- **Injection surfaces** — SQL, HTML, shell, path, header injection paths reviewed.
- **Audit / logs** — mutations leave a trace where the design called for it.
- **Secrets** — no credentials/tokens/PII echoed in responses or error messages.
- **Error messages** — no stack traces, internal paths, or system-internal field names leaked.
- **Rate limit / quota** — new surfaces inherit the family's policy.

Do not paste the full menu blindly. Pick the relevant items; drop the rest.

---

## Red Flags

Stop and self-correct if you notice any of these mid-execution:

- Checklist item phrased as "implement X" / "do the Y" without naming a file, symbol, or specific area.
- A phase has zero concrete tasks — re-think the phase shape.
- About to add a checklist item that doesn't trace to any `D{n}`.
- Force-fitting an example pattern instead of reasoning phases from the task.
- Standalone phase for **unit** tests. (Integration / e2e / contract / load phases are fine when the task warrants them.)
- About to start coding ("and now run it") — refuse and hand off.
- Summary table has rows pointing at files the Phase 2 code map didn't cover.

---

## Anti-rationalizations

| Excuse | Rebuttal |
|---|---|
| "The user said 'just give me a quick plan' — I can skip Sources / Overview / Summary." | No. Universal sections aren't optional just because the task is small. Skipping them ships unreviewable plans. |
| "I already understand the codebase — Phase 2 mapping is overhead." | Run the mapping anyway. Cost of guessing wrong file names is paid by every future reader. |
| "The design doc has one minor open question; I'll proceed with a reasonable assumption." | Stop and route back to `design-doc`. Plans built on un-resolved decisions are worse than no plan. |
| "Phase shape is obvious — I'll skip stating it explicitly." | State it inline at the top of the impl doc. Reviewers need to challenge the shape choice. |
| "I'll write checklist items as 'implement X' and refine during execution." | Refine *now*. Vague items are unactionable. File / symbol / area named upfront. |
| "Tests are obvious — I'll skip the test plan section." | If per-layer tests need documenting (e.g. backend has unit + e2e + contract), include it. Otherwise note "tests inline in checklist" explicitly — don't leave it implicit. |
| "I'll add the security pass to be thorough" / "I'll skip — this is a small change." | Trigger must fire concretely (external surface, authz, user input, audit-relevant state, secrets/PII). "Thorough" is not a trigger; "small" is not an exemption. Run the triggers, include only what fires. |
| "The design didn't cover this fork, but I'll pick a reasonable option to keep momentum." | No. Stop, update the design doc, then resume the impl plan. The impl doc never out-runs the design. |
| "The user asked me to 'and now run it'." | No. Execution is a separate session. Hand off explicitly with the doc path and a one-line statement that this session is plan-only. |
| "I'll add a standalone Tests phase to be thorough." | Default: fold **unit** tests into the phase that builds what's tested. Standalone phase only for heavyweight cross-cutting suites (integration, e2e, contract harness, load) when the task warrants them. |
| "Phase 5 verification is overhead — the doc looked complete." | Run all six checks anyway. "Looked complete" is the rationalisation that ships unreviewable plans. |
| "The user trusts me to skip steps." | They trust the workflow contract: Plan → Verify → Hand off. Skipping Verify is breach of contract. |
| "I found a refactor adjacent to the task; I'll add a phase for it." | Out of scope. Surface the suggestion in chat as a *separate* potential design doc; do not smuggle it into this impl doc. |
| "I'll forge ahead and forget the Sources block — I can add it at the end." | Add it first. Sources is the contract: this impl doc encodes *those* design doc(s) and nothing else. |

---

## Reference

- `template.md` — markdown skeleton: universal layout + optional section shapes.
- `design-doc` skill — produces the resolved design doc that `design-task` consumes.
- Execution — happens in a separate session/agent using this doc as the source of truth.
