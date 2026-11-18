# Final-Form Design Doc — Markdown Skeleton

Pure structural reference. SKILL.md decides which optional sections to include and when; this file only shows the markdown shape of each section.

---

## 1. Universal skeleton

```markdown
# <Subject> — Design Document

<one-sentence intro: what this doc decides / documents>

> **Status**: <see §2>

## Context

<2–6 sentences, anchored in code: what this is, why it needs a doc, what's in/out of scope>

## D1: <terse title> — RESOLVED → <chosen option>

<decision section — see §3>

## D2: <terse title> — RESOLVED → <chosen option>

...

## Summary

<one lead line: the outcome in plain words — what ships, or what was decided>

- **<Decision label>:** <one or two sentences> (D1)
- **<Decision label>:** <one or two sentences> (D2, D3)
- **Key insight:** <the finding that shaped the plan>
- **Tradeoff:** <what was given up, and why it's acceptable>
- **Rollout:** <only if it matters: flag, order, phases>   <!-- optional -->
- **Open:** <only if anything is open: O{n} / gate>         <!-- optional -->
```

---

## 2. Status header forms

```
> **Status**: DRAFT — under design
> **Status**: RESOLVED
> **Status**: RESOLVED — applied to `<symbol/file>`
> **Status**: PARTIAL — phase 1 implemented, phase 2 pending
```

More header lines can follow the Status line:

```
> **Status**: RESOLVED — applied to `<repo>#<PR>`
> **History**: amended <date>: <what changed>
> **Source**: <where the requirements came from>
```

- **One fact per line:** Status holds the state and where it was applied. History and source get their own lines.
- **Line breaks:** end every line except the last with two spaces. Without them, Markdown joins the lines into one paragraph.

---

## 3. Decision section shape

```markdown
## D{n}: <terse title> — RESOLVED → `<chosen option summary>`

### Context

<2–4 sentences. What was the question? What in the code surfaced it? Reference file:line.>

### Decision: Option <X> (<name>)

<One paragraph: the resolution and the one governing reason.>

<Optional: bullet-form supporting rationale, each anchored to code.>

<Optional: small example (struct, JSON, pseudocode) showing the chosen shape.>

### Options evaluated

| Option | Shape | Why not |
|---|---|---|
| A. <name> (chosen) | `<shape>` | — |
| B. <name> | `<shape>` | <one-line "why not"> |
| C. <name> | `<shape>` | <one-line "why not"> |

### When the rejected option would be needed   <!-- optional sub-section -->

<2–3 sentences.>

### Consequences   <!-- optional sub-section -->

<bullet list of what now changes: validation, error codes, mapper logic, migrations, etc.>
```

---

## 4. Optional section shapes

Each block below shows the markdown for one optional section. The selection criteria live in SKILL.md.

### 4a. Inventory

```markdown
## Inventory

| Field / Concern | <Layer/Context A> | <Layer/Context B> | <Layer/Context C> | Notes |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |
```

### 4b. Implementation status

```markdown
## Implementation status

| Capability | Status | Live reference |
|---|---|---|
| <feature slice> | done | `<symbol>` in `<file>` |
| <feature slice> | pending | <gating reason> |
| <feature slice> | not built | <future plan> |
```

### 4c. Open items

```markdown
## Open items

| # | Item | Gate |
|---|---|---|
| O1 | <one-line description> | <what unblocks it> |
| O2 | <one-line description> | <gate> |
```

### 4d. HISTORICAL

```markdown
## HISTORICAL: <title>

> Pre-redesign / superseded shape. Kept for reference; see D{n} for the current decision.

<old struct, old contract, old endpoint, etc.>
```

### 4e. Terminology

```markdown
## Terminology

| Context | Term | Notes |
|---|---|---|
| <where> | `<term>` | <what it means here> |
```

### 4f. Mapping

Single-table form:

```markdown
## Mapping

| <Representation A> | <Representation B> | Notes |
|---|---|---|
| ... | ... | ... |
```

Two-direction form:

```markdown
## Mapping — A → B

| A | B | Notes |
|---|---|---|

## Mapping — B → A

| B | A | Notes |
|---|---|---|
```

### 4g. Validation rules

```markdown
## Validation

| Field / Condition | Code | Message |
|---|---|---|
| `<path>` | `<code>` | `"<exact user-facing string>"` |
```

### 4h. Implementation buckets

```markdown
## Implementation

### Trivial / already in place
- [x] <item> — <reference>

### Small (self-contained)
- [ ] <item>

### Medium (new code needed)
- [ ] <item>

### Open / blocked
- [ ] <item>
```

### 4i. Phased rollout

```markdown
## Phases

**Phase 1 (current): <one-line goal>**
- <what's in>
- <what's out>

**Phase 2 (future): <one-line goal>**
- <what's added>
- <what's still out>
```

### 4j. Architecture diagram

```
ComponentA
 └─ ComponentB ──→ ComponentC
                     └─ ComponentD
```

### 4k. Cross-doc reference

```markdown
> See [<sibling-doc>.md](<sibling-doc>.md) for <topic>.
```

Inline near where the reference is needed; no dedicated section unless there are many.
