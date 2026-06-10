# Implementation Tracker Doc — Markdown Skeleton

Pure structural reference. SKILL.md decides which optional sections to include and when; this file only shows the markdown shape of each section.

---

## 1. Universal skeleton

```markdown
# <Feature> — Implementation Doc

Consolidated implementation plan for <feature>.
Sources: <design-doc-1>.md (decisions D{n}), <design-doc-2>.md, ...

> **Status:** <see §2>

## How to execute this doc

> Read this before starting any work. This is the contract between the impl doc and whoever picks it up.

1. **This doc is the source of truth.** Every change must trace to a `[ ]` item below. If a change does not map to an item, stop — check whether you have drifted.
2. **Do not tick boxes yourself.** `[x]` means "verified against spec." Only `/design-task-check` ticks boxes (or marks `[~]` for partial / drift). You finish work, then run the check; the check ticks what passes.
3. **Run `/design-task-check` after each impl-doc phase** (not just at the end). It is the verification gate: it reads the code, compares to the source `D{n}`, ticks `[x]` for items that match spec, `[~]` for items with drift, and leaves `[ ]` for items not yet shipped. Iterative invocations are idempotent — already-`[x]` items are skipped on re-runs.
4. **Stop on uncovered design forks.** If an item surfaces a decision the upstream design did not resolve, do not invent an answer here. Drive the new decision through `/design-doc`, then resume.
5. **Stop on unexpected complexity.** If an item turns out to be wrong, blocked, or larger than stated — surface to the user. Update the doc together before continuing.
6. **Stop on adjacent-refactor temptation.** If implementing one item makes you want to touch unrelated files, note as a follow-up. Do not smuggle it in.

## Overview

<2–6 sentences: what feature, what areas/layers touched, key shape>

| Area / Surface / Component | <relevant attribute> | Status |
|---|---|---|
| ... | ... | <Shipped|Planned|Partial> |

## Implementation checklist

> **Status key:** [x] = verified against spec (set by `/design-task-check`), [~] = partial / drift (set by check), [ ] = not started or pending verification

### Phase 1: <name>

- [ ] **1. <task>** — `<file>` / `<symbol>` / `<area>`
- [ ] **2. <task>** — `<file>` / `<symbol>` / `<area>`

### Phase 2: <name>

- [ ] **N. <task>**

...

### Phase N: Doc + status

- [ ] **N. Status flipped** to `SHIPPED`
- [ ] **N. `Live reference` cells populated** in §Overview

## Summary — what changes where

| Layer / Area | File / Module | Change |
|---|---|---|
| <area> | `<path>` | <one-line> |
```

---

## 2. Status header forms

```
> **Status:** DRAFT — ready for execution
> **Status:** IN PROGRESS — phase 4
> **Status:** PARTIAL — phases 1–6 shipped, phase 7 pending
> **Status:** SHIPPED
```

---

## 3. Optional section shapes

Each block below shows the markdown for one optional section.

### 3a. Contracts (typed structs / signatures / component props)

```markdown
## Contracts

### <operation / component / function>

\`\`\`<lang>
<type definition>
\`\`\`
```

### 3b. Field mapping (translation between two representations)

Single-table form:

```markdown
## Field mapping

| <Representation A> | <Representation B> | Notes |
|---|---|---|
| ... | ... | ... |
```

Two-direction form:

```markdown
## Field mapping

### <Representation A> → <Representation B>

| A | B | Notes |
|---|---|---|

### <Representation B> → <Representation A>

| B | A | Notes |
|---|---|---|
```

### 3c. Layer-assignment table

```markdown
## Validation — layer assignment

| Rule | <Layer 1> | <Layer 2> | Source of truth |
|---|---|---|---|
| ... | ✅ | — | <where> |
```

### 3d. Field-error table

```markdown
### Field-error table

`prefix` is `<json.path / form.field / etc.>`.

| Field | Rule | `code` | Source |
|---|---|---|---|
| `<path>` | <rule> | `<code>` | <where it's enforced> |
```

### 3e. Error mapping (internal → external)

```markdown
## Error mapping

\`\`\`<lang>
<framework-specific mapping: internal key → external code/message/field path>
\`\`\`
```

### 3f. Mode / variant table

```markdown
### Modes

| Mode | <fields/state> | <mapping/behavior> |
|---|---|---|
| `<mode>` | `<fields>` | `<mapping>` |
```

### 3g. Phased rollout

```markdown
## Rollout phases

**Phase 1 (current):** <one-line goal>
- <what's in>
- <what's out>

**Phase 2 (future):** <one-line goal>
- <what's added>
```

### 3h. Test plan

```markdown
## Tests

| Layer / Area | Required tests |
|---|---|
| <layer> | <what's covered: rules, paths, edge cases> |
```

### 3i. Security pass

```markdown
## Security pass

- [ ] **<check>** — <one-line description>
- [ ] **<check>** — <one-line description>
```

(Pick checks from the SKILL.md menu; populate only those that fit the task.)

### 3j. HISTORICAL

```markdown
## HISTORICAL: <title>

> Pre-redesign / superseded shape. Kept for reference; see <doc> for the current decision.

<old struct, old contract, old shape, etc.>
```

### 3k. Cross-doc reference

```markdown
> See [<sibling-doc>.md](<sibling-doc>.md) for <topic>.
```

Inline near where the reference is needed; no dedicated section unless there are many.
