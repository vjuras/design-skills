---
name: design-doc
description: Author a feature design document by analyzing existing code first, surfacing every decision point that the code cannot answer alone, and drafting each as options with pros/cons/risks (one marked as recommendation) directly in the doc — then handing off for the user to read and respond. Produces a detailed working-draft form as the deliverable; collapse to the slim final form is a separate, opt-in step (see the design-cleanup skill). Use when the user says "/design-doc", "design doc", "write a design doc", "design <feature>", or asks to document decisions for a feature that exists (or is planned) in code.
---

# Design Doc — Code-First, Decision-Driven

Author design docs through an **analyze → draft all decisions into the doc → user reads and responds** loop. The doc is the workspace; it is also where the user makes decisions. Do **not** drive decisions through chat-side question prompts — the surface is too narrow and decisions need full context.

The detailed working draft (every option, every "why not", every recommendation written out long-form) **is the deliverable** for this skill. Collapsing to the slim final form per `template.md` is a separate, opt-in step owned by the `design-cleanup` skill.

`template.md` is a **markdown shape reference** — it shows what each section looks like rendered. This file (`SKILL.md`) owns the instructions: when to start, what to write into the doc, when to stop, when to hand off to `design-cleanup`.

## When to use

Trigger phrases: `/design-doc`, "write a design doc for X", "design <feature>", "I need a design doc on Y", "document the decisions for Z".

Use when the goal is a written, decision-bearing document about a feature, contract, or refactor — typically **not** when the user just wants a quick implementation plan or a `todo.md`.

**When NOT to use:**
- Single-line fixes, typos, or comment-only edits — no decisions to surface.
- User wants a `todo.md` or quick implementation plan, not a decision-bearing doc.
- Feature is already designed in a sibling doc — point at it instead.
- Pure refactors with no contract or behaviour change — no decisions to surface.

## Core principles (do not skip)

1. **Code first, prose second.** Every claim traces to a `file:line` or symbol, or carries an explicit `<assumption>` flag. No exceptions.
2. **No silent assumptions.** Ambiguity in code → `Q{n}` in `## Open questions`. Never a guess that reads like a fact.
3. **The doc is the deliberation surface.** Options, pros/cons, risks, recommendations all live in the doc. Never drive decisions through chat-side multiple-choice prompts — the surface is too narrow.
4. **Decisions are user-owned.** You recommend; the user decides. You never flip `— PROPOSED` to `— RESOLVED` without an explicit user pick.
5. **The doc is process-shaped, not essay-shaped.** Tables, checkboxes, decision gates over prose. One paragraph per rationale, max. One sentence per bullet. If a section wants to be three paragraphs, it's padding. The reader is about to *do* something — implement, review, follow up — not consume an essay.
6. **Working draft ≠ final form.** The detailed working draft is *this* skill's deliverable. Collapsing to slim final form is `design-cleanup`'s job and is opt-in.

---

## Workflow

### Phase 0 — Detect entry state

If the user invokes this skill on a path that may already exist, read the file first and route to the right phase:

| Doc state | Action |
|---|---|
| File doesn't exist | Phase 1 — frame the doc. |
| Stub (title + intro + status header, no `D{n}` yet, no `## Pending decisions`) | Phase 2 — start code-first analysis. |
| Has `## Pending decisions` and/or `D{n} — PROPOSED` sections | Phase 3 — resume. List the pending `D{n}` IDs and titles in chat with their recommendations, point at the doc, wait for the user. |
| Every `D{n}` is `— RESOLVED → Option <X>`, but `## Open questions` or `## Pending decisions` scratch is still present | **Stop.** This is a cleanup task, not an authoring task. Tell the user: "All decisions are resolved. Run `/design-cleanup` to collapse to final form." |
| Already in final form (no working-draft scratch, status `RESOLVED`) | Stop and ask the user what they want to change — a new decision (Phase 2/3), a status update, or something else entirely. |

For new docs, proceed straight to Phase 1.

### Phase 1 — Frame the doc

Before reading any code, agree with the user on:

- **Subject** — one sentence: "Design doc for X."
- **Scope boundary** — what's in, what's explicitly out. Out-of-scope items get one sentence: "Excluded — see future work."
- **Doc location and filename** — propose a path next to existing docs if a folder convention exists; otherwise ask. Use lowercase, hyphenated.
- **Existing doc family to mirror** (if any) — point at 1–2 sibling docs whose tone/structure you'll match. If none exist, default to `template.md`.

Write a stub file with just the title, one-line intro, and `> **Status**: DRAFT — under design`. Everything else accumulates in the working draft as the loop progresses.

### Phase 2 — Code-first analysis

**Goal:** understand the feature from code well enough to surface every place behaviour is unobvious, layers disagree, or the contract has multiple valid shapes.

For non-trivial codebases, **delegate exploration to parallel sub-tasks / sub-agents** if your environment supports them — this keeps main context clean. Run one per layer/area when the feature spans concerns:

- "Map every place `<feature>` appears in `<layer/area>`. Return file:line references and a field-by-field / behaviour-by-behaviour summary."
- One sub-task per layer/area is a good default (e.g. domain, persistence, runtime, UI, API). If parallel execution isn't available, sequence them.

**While reading, log two lists in the working draft:**

```markdown
## Open questions
- [ ] Q1: <question>. Source: <file:line>. Why it matters: <one sentence>.

## Pending decisions
- [ ] D1: <one-line summary of what needs to be decided>
```

Open questions are things only the user (or another teammate) can answer (e.g. "Is field `X` actually used in production, or is it vestigial?"). Pending decisions are forks where the code allows multiple valid answers.

**Scope-discipline gate.** For every candidate decision surfaced during analysis, run this filter before adding it to `## Pending decisions`:

| # | Question | If no |
|---|---|---|
| 1 | Inside the scope boundary set in Phase 1? | Add to a `## Out of scope` list with a one-line reason. Do not draft a `D{n}`. |
| 2 | Cost of being wrong non-trivial? (Would two reasonable engineers pick differently?) | It's a default, not a decision. Note the default in the relevant `Context`; do not draft a `D{n}`. |
| 3 | Decidable on its own, or does it ride along with another `D{m}`? | If it rides along, fold into `D{m}.### Context` as a note. Do not draft a separate `D{n}`. |

Only candidates that pass all three gates earn a `D{n}`. "While I was reading the code I found another decision worth making" is the smell — run it through the gate before drafting.

### Phase 3 — Draft every decision into the doc

For each item in `## Pending decisions`, expand into a `D{n}` section in the working draft using this exact shape:

```markdown
## D{n}: <terse decision title> — PROPOSED

### Context
<2–4 sentences. What's the question? Why does it need answering? Reference the code (file:line) that surfaced it. If this decision depends on another (`D{m}`), say so explicitly here — including how each plausible resolution of `D{m}` would change the option set below.>

### Options

| Option | Shape (one line) | Notes |
|---|---|---|
| A. <name> | `<concise representation>` | <one-line note> |
| B. <name> | `<concise representation>` | <one-line note> |
| C. <name> | `<concise representation>` | <one-line note> |

### Option A — <name>

<small code/sketch if useful>

**Pros**
- <bullet>

**Cons**
- <bullet>

**Risks** (if any)
- <bullet>

### Option B — <name>
<same structure>

### Option C — <name>
<same structure>

### Recommendation: Option <X>
<One paragraph. Which option to recommend and why — be specific. If no option is clearly stronger, say so and explain what would tip the choice (e.g. "depends on whether <X> matters more than <Y>").>

### Awaiting decision from user.
```

**Length discipline (per `D{n}`).** Keep the produced doc process-shaped:
- `### Context`: 2–4 sentences.
- Each Pros / Cons / Risks bullet: one sentence. If a bullet wants two, split or move the second to `### Context`.
- `### Recommendation`: one paragraph, max. If you need more, you're rationalising — distill to one governing reason.

**Draft all currently-known decisions in one pass** so the user sees the whole shape before responding. For dependent decisions, name the dependency in `### Context`; if the option set genuinely forks on another `D{m}`, list the conditional options under each branch.

When the batch is drafted, **hand off to the user**. Do not drive decisions via interactive multiple-choice prompts — the surface is too narrow and the user can't compare pros/cons/risks side-by-side. Instead, post a short chat message:

- Path to the doc.
- A bullet list of every `D{n}` with its title and your recommendation in one line each (e.g. `D1: Field naming — recommend Option B (snake_case)`).
- A single sentence inviting the user to read the doc and respond however suits them: reply in chat (`D1→A, D2→B with my reasoning…`), edit the doc directly, or leave inline comments.

Then stop and wait. Do not flip any `— PROPOSED` to `— RESOLVED` until the user has explicitly chosen.

When the user responds, edit each affected section in place — but **lightly**. Keep the full options bodies (the long-form Pros/Cons/Risks for every option). Collapse to the slim final form is `design-cleanup`'s job, not this skill's.

Resolution edits per `D{n}`:
- Change heading from `— PROPOSED` to `— RESOLVED → Option <X>`.
- Update the `### Recommendation: Option <X>` heading to `### Decision: Option <X> (<name>)` and refine the paragraph if the user added reasoning the recommendation didn't capture (verbatim where they gave wording).
- Mark the chosen row in the `### Options` table with a `← chosen` note (or rename column header to `Chosen?` and tick it). Leave the rest of the table and every option's full body in place.
- Delete the `### Awaiting decision from user.` line.
- Tick the item in `## Pending decisions`.

If the user picks something *not* in your option list, add it as a new option (with full Pros/Cons/Risks captured from their reasoning verbatim), then mark it chosen. If the user resolves only some decisions and asks to defer others, leave the deferred ones as `— PROPOSED` and continue.

### Phase 4 — Iterate until the queues drain

Loop Phase 2 ↔ Phase 3:
- New questions surface as decisions land → add them to the queues, draft them as a fresh batch into the doc, hand off again.
- If a decision invalidates an earlier one, mark the earlier `— SUPERSEDED by D{m}` rather than deleting it.
- If the user's resolution of `D{m}` collapses the option set of a still-`— PROPOSED` `D{n}`, edit `D{n}` in place to reflect the now-known branch (don't silently resolve it — the user still picks).

**Stop conditions:**
- `## Open questions` and `## Pending decisions` are empty (all answered or explicitly deferred).
- Every `D{n}` is `— RESOLVED → Option <X>`. The detailed working draft is the deliverable; proceed to Phase 5.
- The user says "good enough, ship it" — capture any remaining items as inline notes for cleanup to fold into `Open items`.

### Phase 5 — Verify before announcing done

Before telling the user the doc is ready, run these checks. Evidence required — "looks right" is not sufficient.

1. **No `— PROPOSED` left.** Zero occurrences of `— PROPOSED` in the doc.
2. **No unticked checkboxes in working-draft scratch.** Every `- [ ]` item in `## Open questions` and `## Pending decisions` must be ticked, removed, or carry an explicit deferral note.
3. **Every `D{n}` has a `### Decision` heading and a one-paragraph rationale.** Read each `D{n}`. If the Decision paragraph is missing or runs longer than one paragraph, fix it. Bullets one sentence each.
4. **Every `### Context` paragraph cites code.** Each must contain a `path/to/file.ext:line`, a `` `<symbol>` `` reference, or an explicit `<assumption>` flag. Anything else is a silent assumption — fix or flag.
If any check fails, fix and re-run. Only when all four pass, post the final one-liner: *"All decisions resolved. The detailed doc is at `<path>`. If you want the slim final form, run `/design-cleanup`."* Do **not** auto-run cleanup.

---

## Heuristics — what counts as a "decision point"

You are looking for forks where the code does not commit to a single answer. Common shapes:

- **Naming divergence** between layers or contexts.
- **Field used by some callers, ignored by others** — expose, drop, or document?
- **Multiple valid shapes** for the same concept (flat vs. discriminated; enum vs. boolean; array vs. single).
- **Lifecycle / state model** that's currently implicit and needs to be made explicit.
- **Side effects on update / delete** that have to be made explicit in the contract.
- **Validation placement** — surface vs. depth.
- **Read-only vs. read-write** scope for a field that exists.
- **Legacy / feature-flagged data** that has to round-trip but should not be createable.
- **Security & exposure** — auth scope per endpoint, ownership rules for referenced IDs (does the consumer's brand own this template/segment?), redaction of sensitive fields, read-only enforcement on server-controlled state. These have to be designed, not bolted on at implementation time.

(Decisions that boil down to "match the existing convention with no friction" are defaults, not decisions — the Phase 2 scope-discipline gate filters them out.)

## Heuristics — question vs. decision

| Symptom | Treat as |
|---|---|
| Code says one thing, comment says another | Question (read again; ask if still ambiguous) |
| Two callers of the same function pass different shapes | Question |
| Field exists in struct but no callsite reads it | Question ("is this dead?") then maybe Decision ("expose or drop?") |
| Two reasonable shapes both round-trip cleanly | Decision |
| Behaviour depends on environment, brand, or feature flag | Question (gather facts) before Decision |

## Tone (working draft)

(Core principle #5 covers length and shape — tables over prose, one paragraph per rationale, one sentence per bullet. Conventions specific to this skill:)

- Third person, declarative. No "we" / "I". No throat-clearing ("In this section…") — get to the table.
- Cite code by `path/to/file.ext:line` or symbol; quote a block only if its shape is structurally informative.
- Code identifiers in backticks always — fields, files, types, constants.
- `>` blockquote is for status indicators and cross-doc references only. The Status line carries the state and where it was applied; amendment history goes on its own `> **History**:` line, per `template.md §2`. End every header line except the last with two spaces, or Markdown joins them into one paragraph.
- Most decision-bearing docs land at 100–600 lines. Past 1000 is a smell — split into parent + sub-docs.

## Red Flags

Stop and self-correct if you notice any of these mid-execution:

- Drafting a `D{n}` whose `### Context` cites no `file:line` and carries no `<assumption>` flag.
- More than ~5 `D{n}` in a single batch — likely over-decomposing; re-run the Phase 2 scope-discipline gate.
- About to drive a decision via an interactive multiple-choice prompt instead of writing it into the doc.
- About to flip `— PROPOSED` to `— RESOLVED` without an explicit user pick.
- Recommendation paragraph running longer than one paragraph.
- Searching the same file twice in Phase 2 for a named symbol — the code map is probably wrong.

---

## Anti-rationalizations

| Excuse | Rebuttal |
|---|---|
| "I can't find a `file:line` for this; I'll describe it informally." | Either re-search (Grep / subagent) or flag the sentence as `<assumption>` inline. Citation-less prose corrupts the audit trail. |
| "Phase 1 framing is overhead; I already know what the user wants." | Frame anyway. 90 seconds of subject + scope + path + sibling-doc prevents 30 minutes of redirection. |
| "While reading the code I found another decision worth making." | Run it through the Phase 2 scope-discipline gate. Out of scope → defer. Drift kills mergeable docs. |
| "This decision is obvious; I'll mark it RESOLVED to save the user a step." | Recommendation ≠ decision. The user owns the status flip, even when your recommendation is strong. |
| "These two decisions look similar; I'll merge them into one `D{n}`." | Two `D{n}` with cross-refs is cheaper than one bloated section smuggling two forks. Different option sets ⇒ different decisions. |
| "Resolving D2 before D1 is fine — they look independent." | If they truly are, fine. If `D2`'s option set forks on `D1`, name the dependency in `D2.### Context` and let the user pick `D1` first. Never silently assume the upstream resolution. |
| "The user said 'go with B'; I'll write the rationale / polish their terse reply." | Capture user reasoning verbatim, or ask for it. Inventing or polishing falsifies the audit trail. |
| "All options have similar pros; I'll skip the Recommendation paragraph." | Then say so: *"No clear winner — would tip on X."* That is itself a recommendation. |
| "I'll write the rationale as a few paragraphs for clarity." | One paragraph, max. Bullets: one sentence. The doc is process-shaped, not essay-shaped. |
| "Driving decisions via an interactive prompt would be faster." | The surface is too narrow — options are stripped to labels, the user can't compare pros/cons/risks side-by-side. Decisions live in the doc, full stop. |
| "I'll add a `## Summary` now to save cleanup time later." | No. Pre-collapsing is `design-cleanup`'s job. The user may never want it collapsed. |
| "I'll delete the rejected options' bodies — the decision is made." | Keep them. Collapse to "why not" rows is `design-cleanup`'s job. Authoring-time deletion destroys the audit trail. |
| "Open questions all answered in chat — no need to record." | Record the answer in the relevant `D{n}.### Context` or as a deferred follow-up. Chat is not durable. |
| "The user said 'good enough, ship it' — I can skip Phase 5 verification." | No. Run the four checks anyway. The user authorising shortcuts is the most common path to broken docs. |

---

## Reference

- `template.md` — pure markdown skeleton: universal layout, status header forms, decision section shape, and the ten optional section shapes. No instructions in that file — only structures.
- `../design-cleanup/SKILL.md` — companion skill that collapses the detailed working draft into the slim final form per `template.md`. Run it when (and only when) the user wants the distilled artifact.
