---
name: design-cleanup
description: Collapse a detailed working-draft design doc into final form per template.md — remove the open-questions / pending-decisions scratch, fold each D{n} into the resolved-decision shape with rejected options table-ified, pick optional sections via the selection table, and flip the status header. Use when the user says "/design-cleanup", "finalize this design doc", "collapse to final form", or invokes /design-doc on a doc whose decisions are all already RESOLVED.
---

# Design Cleanup — Collapse Working Draft to Final Form

The detailed working draft (every option, every recommendation, every "why not" written out long-form) is itself a valuable artifact and may be the user's preferred end state. Cleanup is **opt-in distillation** — turn the draft into the slim final form per `../design-doc/template.md` only when the user asks for it.

This skill is purely mechanical. It does **not** make new decisions, re-analyze code, or rewrite prose for style. If a decision is still `— PROPOSED`, this skill stops and points back at `design-doc`.

## When to use

Trigger phrases: `/design-cleanup`, "finalize this design doc", "collapse to final form", "distill this design doc", "produce the final form".

Also auto-routes here: `/design-doc` invoked on a doc where every `D{n}` is `— RESOLVED → Option <X>` and working-draft scratch is still in place.

**When NOT to use:**
- Any `D{n}` is still `— PROPOSED` — route to `/design-doc` to resolve it first.
- Open questions remain unanswered — they must land in `D{n}.### Context` or `Open items`, not be silently dropped.
- User prefers the detailed working draft as the deliverable — cleanup is opt-in distillation, not mandatory.

## Core principles (do not skip)

1. **Mechanical, not editorial.** `### Context` and `### Decision` paragraphs are preserved verbatim. Cleanup never rewrites prose for style.
2. **Stop if any `D{n}` is `— PROPOSED`.** Route back to `design-doc`. Cleanup never decides.
3. **Every rejected option keeps its own row.** Collapsing two rows because their "why nots" are similar destroys the audit trail.

---

## Input contract (validate before touching anything)

The doc must satisfy **all** of:

1. File exists and has a status header (e.g. `> **Status**: DRAFT — under design`).
2. Every `## D{n}: <title> — …` heading is `— RESOLVED → Option <X>` (no `— PROPOSED` left).
3. `## Pending decisions` (if present) — every checkbox is ticked.
4. `## Open questions` (if present) — every item is either explicitly answered in the relevant `D{n}.### Context` or moved to a follow-up note for the `Open items` section.

If **any** check fails, **stop**. Post a short message listing which `D{n}` are still PROPOSED (with file:line) or which open questions remain, and tell the user to run `/design-doc` to drive them. Do not collapse a partially-resolved doc.

## Workflow

### Phase 1 — Validate

Read the doc. Run the four input-contract checks above. If validation fails, stop with a clear failure message. If it passes, continue.

### Phase 2 — Mechanical collapse

For each `D{n}`, transform the working-draft shape into the resolved shape from `../design-doc/template.md §3`:

**Working-draft shape (input — what `design-doc` produces upon resolution):**
```markdown
## D{n}: <title> — RESOLVED → Option <X>

### Context
…

### Options
| Option | Shape | Notes (chosen?) |
…

### Option A — <name>
…Pros / Cons / Risks…

### Option B — <name>
…Pros / Cons / Risks…

### Decision: Option <X> (<name>)
<one-paragraph rationale, possibly with user-added reasoning>
```

**Final shape (output):**
```markdown
## D{n}: <title> — RESOLVED → `<chosen option summary>`

### Context
<2–4 sentences, code-anchored, unchanged from working draft>

### Decision: Option <X> (<name>)
<rationale paragraph, unchanged from working draft>

<optional: small example/sketch from the chosen option's body>

### Options evaluated
| Option | Shape | Why not |
|---|---|---|
| A. <name> (chosen) | `<shape>` | — |
| B. <name> | `<shape>` | <one-line distilled from the cons/risks of B> |
| C. <name> | `<shape>` | <one-line distilled from the cons/risks of C> |

### Consequences   <!-- include only when non-trivial -->
- <bullets distilled from chosen option's risks + downstream implications>
```

Mechanical rules:
- `### Context` and `### Decision` paragraphs are preserved verbatim. Cleanup is not editorial.
- The chosen option's `Pros/Cons/Risks` body folds into the `Consequences` section (risks + non-obvious implications) when non-trivial. The Pros are already in the `Decision` paragraph.
- Each rejected option's `Pros/Cons/Risks` body collapses to **one** "Why not" line in the `Options evaluated` table — the single most decisive reason against. Delete the per-option `### Option X — <name>` subsections after distilling.
- The original `### Options` table (with the `← chosen` marker) is replaced by the `### Options evaluated` table. Same options, new column shape.
- If a `D{n}` is marked `— SUPERSEDED by D{m}`, leave it in place; do not collapse.

### Phase 3 — Remove scratch and add structural sections

- Delete `## Open questions` entirely (every item should already be answered or moved).
- Delete `## Pending decisions` entirely.
- Add `## Summary` at the bottom in the shape of `../design-doc/template.md §1`: the outcome, the key decisions, the key insight, the intentional tradeoff. Not a recap.
- Pick optional sections from `../design-doc/template.md §4` using the **selection table** below. Skip any section whose trigger does not fire.

### Phase 4 — Flip status

Update the status header:
- All decisions resolved, code not yet changed → `> **Status**: RESOLVED`
- All decisions resolved, code has landed → `> **Status**: RESOLVED — applied to <symbol/file>`
- Some phases shipped, others pending → `> **Status**: PARTIAL — phase 1 implemented, phase 2 pending`

### Phase 5 — Verify before handing off

Before announcing the cleaned doc, run these checks. Evidence required — "looks right" is not sufficient.

1. **No working-draft scratch sections remain.** No `## Open questions` or `## Pending decisions` headings in the doc.
2. **No leftover handoff scaffolding.** Zero occurrences of the `Awaiting decision from user.` line.
3. **No per-option subsections left.** No `### Option A —`, `### Option B —` (etc.) headings remain. Each `D{n}` should have `### Decision`, `### Options evaluated`, and (optionally) `### Consequences`.
4. **Every `D{n}` has the resolved shape.** Read each one. Confirm `### Context` (code-anchored), `### Decision: Option <X> (<name>)` (one-paragraph rationale), and `### Options evaluated` table with all options as rows.
5. **Status header reflects reality.** Matches one of the three forms in Phase 4.
6. **`Open items` (if present) has the `Gate` column populated for every row.** No empty Gates.
7. **Summary captures the punchline.** Its first line and bold labels alone give the point. Not a recap.

If any check fails, fix and re-run.

When all seven pass, post a short chat message summarising what changed: number of `D{n}` collapsed, lines of scratch removed, optional sections included (list them by name), and any judgment calls flagged for the user to review (e.g. *"included `Mapping` section because the doc translates between request DTO and domain entity — confirm or drop"*).

The user reviews the final form in the doc, not in chat. If they want adjustments, they say so and you re-run the relevant phase.

---

## Picking optional sections

`../design-doc/template.md §4` lists optional section shapes. Include each only when its trigger fires. If no trigger applies, the doc has only universal sections — that is fine and often correct.

| Optional section (`template.md §`) | Include when |
|---|---|
| `4a. Inventory` | Feature spans multiple layers/components/contexts AND those layers diverge in name, semantics, or coverage. Skip if everything maps 1:1. |
| `4b. Implementation status` | Feature is partially shipped (some slices live, some pending, some not built). Skip if the doc is purely forward-looking or fully shipped in one pass. |
| `4c. Open items` | The doc ships with known unresolved follow-ups that have a clear gating condition. Skip if there are no follow-ups. |
| `4d. HISTORICAL` | A prior shape, decision, or naming is likely to be referenced or re-asked (pre-redesign struct still visible in production data; rejected approach still present in old code). Use sparingly — one-line "why not" rows in the Options table cover most cases. |
| `4e. Terminology` | The same concept goes by different names across layers, teams, or sibling docs and the divergence is non-obvious. |
| `4f. Mapping` | The topic is translation between two representations (external contract ↔ internal model, request DTO ↔ domain entity, legacy schema ↔ new schema). Use the two-direction form when defaults, dropped fields, or detection logic differ between directions. Skip entirely when the topic is purely internal. |
| `4g. Validation rules` | The topic defines a contract surface that rejects malformed input. Skip on refactors, internal restructures, performance work. |
| `4h. Implementation buckets` | The doc is meant to drive sequenced rollout work. Skip when the doc is purely descriptive or post-hoc rationale. |
| `4i. Phased rollout` | The feature ships in distinct phases with different contracts. Skip when shipping in one go. |
| `4j. Architecture diagram` | The orchestration layout is non-obvious from the section structure alone. Don't reach for diagrams when a table or bullet list is clearer. |
| `4k. Cross-doc reference` | Inline near where the reference is needed; no dedicated section unless there are many. |

**Default posture:** start with universal sections only (Context, `D{n}`s, Summary). Add an optional section only when its trigger fires. Empty or filler sections are worse than missing ones.

---

## Red Flags

Stop and self-correct if you notice any of these mid-execution:

- About to edit a `### Context` or `### Decision` paragraph for style — cleanup is mechanical, not editorial.
- Merging two rejected options' rows because their "why nots" are similar — each option keeps its own row.
- About to add an optional section whose trigger didn't fire.
- Per-option `### Option X — <name>` subsections still present after Phase 2 collapse.
- About to flip the status header without re-running the input-contract checks.

---

## Anti-rationalizations

| Excuse | Rebuttal |
|---|---|
| "Most decisions are RESOLVED — I'll collapse the rest by inferring user intent." | If any `D{n}` is `— PROPOSED`, stop and route to `design-doc`. Cleanup never decides. |
| "The user wrote a long Context paragraph; I'll tighten it during collapse." | Cleanup is mechanical. Context is preserved verbatim. Editing for style is editorial drift. |
| "Two rejected options have similar 'why nots'; I'll merge them into one row." | Each rejected option keeps its own row in `Options evaluated`. The audit trail is the value. |
| "The selection table says I might need `Validation rules`; I'll add it just in case." | Trigger must fire concretely. Empty or filler sections are worse than missing ones. |
| "I'll generate a `Summary` paragraph from what I know." | Summary is the punchline (outcome + key insight + tradeoff), one line plus short labelled bullets. If you can't write that confidently, ask the user. |
| "The doc is short after collapse — I'll bulk it up with `Open items` / `Consequences`." | Length is not the goal. Each section earns its keep or it goes. |
| "I'll keep one Pros/Cons/Risks subsection as a 'representative' example." | All rejected options collapse to one-line `Why not` rows. Per-option subsections are deleted. |
| "The original doc had `<assumption>` tags; I'll leave them tagged in the final form." | Final form must either resolve assumptions (with citation) or move them to `Open items` with a `Gate` column. Untracked assumptions in the final form is a regression. |
| "Phase 5 verification is overhead — the collapse looked clean / the user trusts my judgment." | Run all seven checks anyway. The user trusts the *workflow* (validate → collapse → verify), not your sense of cleanliness. |

---

## Reference

- `../design-doc/template.md` — pure markdown skeleton: universal layout, status header forms, decision section shape, and the ten optional section shapes.
- `../design-doc/SKILL.md` — companion skill that authors the working draft and drives decisions to RESOLVED.
