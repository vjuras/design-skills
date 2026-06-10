---
name: design-task-check
description: Verify an implementation tracker doc against the actual code and the upstream design doc(s). Reads each unverified item, compares its code to the source D{n} decision (rule / shape / error code / side effect / algorithm), ticks `[x]` only on full match, marks `[~]` on drift with concrete divergence notes, and leaves `[ ]` for not-yet-started work. Idempotent — already-verified items are skipped on re-runs, so the skill is run incrementally after each impl-doc phase. Cross-checks design alignment (every D{n} has impl coverage, no smuggled-in scope), flags stale Summary rows, and proposes status updates. Use when the user says "/design-task-check", "check progress on this impl doc", "review what's been built", "verify implementation against design", or invokes this skill after finishing an impl-doc phase.
---

# Design Task Check — Verify Implementation Against Plan

Take an implementation tracker doc (produced by `design-task`) plus the upstream design doc(s) it sources from. Verify that what the code actually does matches what was planned and what was decided. (Core principles below codify the contract — this skill is the only thing that ticks boxes, and it never edits production code.)

## When to use

Trigger phrases: `/design-task-check`, `/check-impl`, "check progress on this impl doc", "verify implementation against design", "review what's been built", "did we ship what we planned", "mark progress".

The natural invocation cadence is **after each impl-doc phase** (P1 done → check; P2 done → check; etc.). Each run is incremental: already-`[x]` items are skipped. The user can also invoke at PR review time for a final full pass.

If the user explicitly asks for a full re-verification (*"re-check everything from scratch"*), drop the idempotency rule and re-verify all items including `[x]` ones. Default is skip.

**When NOT to use:**
- Any source design doc is still `DRAFT` or has unresolved `D{n}` — route back to `design-doc` first; the spec is unstable.
- No impl doc exists to verify against — this skill verifies an existing tracker, it does not author one.
- User wants the code modified — this skill is read-mostly on code. Drift gets surfaced; fixing is a separate session.

## Core principles (do not skip)

1. **The check ticks boxes; the engineer does not.** `[x]` means verified by this skill; trust existing `[x]`s unless user explicitly requests full re-verify.
2. **The doc is the spec.** Code disagrees with impl doc → code is the bug; never silently rewrite the impl doc to match drifted code.
3. **Bidirectional alignment.** Every `D{n}` traces to at least one impl item; every impl item traces to a `D{n}` or structural consequence.
4. **Idempotent across runs.** Already-`[x]` items are skipped; trust the previous verdict unless user explicitly asks for full re-verify.
5. **Read-mostly on code.** Update the impl doc; do not edit production code. Surface drift and hand off.
6. **The report is process-shaped.** Tables only (newly verified / drifted / not started / missing coverage / smuggled scope / stale rows), not narrative.
7. **Verification gates the status flip.** No `SHIPPED` without all items `[x]` and all Phase 3 checks passing.

---

## Workflow

### Phase 0 — Locate inputs

1. Identify the impl doc path. If the user invoked `/design-task-check` without specifying, ask: "Which impl doc?" Or scan the current branch's most-recently-modified `*-impl.md` / `*-task.md` files and propose.
2. Read the impl doc. Extract the `Sources:` block — these are the design doc(s) it traces to.
3. Read each design doc. Extract every `D{n}` heading and its resolved option.
4. Confirm scope with the user if the impl doc references design docs the user didn't expect (or vice versa).

If any source design doc isn't `RESOLVED`, stop. Tell the user the design has gaps; route back to `design-doc`.

### Phase 1 — Verify implementation correctness (the load-bearing check)

For each item in the impl doc's `## Implementation checklist`:

**Already `[x]`** — skip. Already verified in a previous run. Trust the previous result. (The user may explicitly ask for a full re-verification, e.g. *"re-check everything from scratch"*; default is skip.)

**`[~]` or `[ ]`** — run this 4-step procedure:

1. **Identify the source decision.** Trace the item back to a specific `D{n}` (or a structural consequence of one) in the design doc(s). Quote the `### Decision` paragraph and any concrete details (rule, shape, error code, side effect, algorithm).

2. **Read the actual content** of the named symbol, file, or artifact — *or the symbol/artifact filling the same role if it was renamed/moved during implementation.* The impl-doc name is a role label, not a literal spelling requirement; renames are a normal part of impl. If you find a role-equivalent under a different name, proceed with steps 3–4 and write the actual locator in `Live reference`. Not just "does it exist" — read the implementation, the migration SQL, the runbook prose, the alert/config YAML. The artifact is whatever the impl item points at (symbol in a code file, a whole config file, a section of a runbook). Use whatever file-reading mechanism your environment provides; do not skip this step.

3. **Verify behaviour matches the decision.** Pick the question that fits the decision shape:
   - **Rule** — Does the code enforce the exact rule? (e.g., decision said `<= 100`, code says `<= 100`, not `< 100`.)
   - **Observable shape** — Does the externally-visible contract match? (JSON tags, exported field names, public function signatures, return types crossing package boundaries.) Exact match required.
   - **Internal shape** — Does the internal layout fill the same role? (Private fields without JSON tags, package-private types, internal helpers.) Role-equivalence is sufficient; renames are not drift.
   - **Error code/message** — Does the code emit the exact code/message string the decision named?
   - **Side effect** — Does the code perform the side effect the decision required? (Audit log, cache invalidation, dual-write, event emission.)
   - **Algorithm/flow** — Does the code follow the algorithm in the right order, with the right edge-case handling?
   - **Config / artifact content** — For non-code items (migration SQL, alert YAML, runbook section, Helm values), does the file's content match the decision verbatim where the decision was specific (column type, threshold value, rollback step)?

4. **Verify tests cover the decision.** Not "tests exist" — would any existing test fail if the decision's specific behaviour broke? Parameterized tests, table-driven tests, and property tests covering the relevant input range count, even if no test explicitly names the boundary value. The check is "does the test surface catch regression," not "does a test mention the boundary verbatim." If the test plan section names tests, locate them. If tests-inline was the choice, scan the named area. Don't run the full suite — just confirm the relevant tests exist.

**Outcome rules (decisive — no rationalising):**

| Result | Action |
|---|---|
| All four steps pass | Tick `[x]`. Fill `Live reference` with the most precise locator: `<symbol> in <file>:<line>` for code, `<file>:<line-range>` for a config/migration block, or `<file>` for a whole-file artifact (runbook, alert YAML). |
| Code present, but at least one step fails | Mark `[~]`. Add a row to the **Drifted** table with the verbatim decision quote, the code citation, the specific divergence, and the `Category` (see below). |
| Code absent (file or symbol not yet present) | Leave `[ ]`. Not started — not a failure, not drift. |

Silent absorption ("most of it works, I'll tick anyway") is the failure mode this phase exists to prevent. If any of the four steps fails, the box stays `[~]` until the next run after a fix.

**Drift classification rubric.** When the 4-step procedure surfaces a mismatch, classify before logging:

| Mismatch kind | Drift? |
|---|---|
| Private symbol/field/local renamed, behaviour preserved | no — update `Live reference` to actual name |
| Different algorithm, same observable result | no |
| Internal package layout shuffled, no public surface change | no |
| JSON tag, exported name, error code emitted, public API path, DB column name, config key, log/event field name mismatch | yes |
| Rule boundary (`<=` vs `<`, threshold value, regex change) | yes |
| Missing required side effect (audit log, dual-write, event emission, cache invalidation) | yes |
| Behaviour matches, no visible test covers the decision-specific behaviour | no — add to Drifted table with `Category: missing test`; the test checklist item (if any) tracks this as its own `[ ]` |
| Tests pass but exercise wrong inputs / wrong boundary | yes |

Default for ambiguous cases: surface to user with the divergence cited; do not silently classify as "not drift."

This phase produces three tables for the report:

**Newly verified** (items flipped to `[x]` this run):

| Item | D{n} | Live reference |
|---|---|---|
| 5 | D2 | `validateName` in `validator.go:42` |
| 8 | D4 | `migrations/0042_add_owner_id.sql` |
| 14 | D7 | `helm/values.yaml:88-95` (alert threshold) |

**Drifted** (items now `[~]`):

| Item | D{n} | Decision (verbatim) | Code (file:line) | Divergence | Category |
|---|---|---|---|---|---|
| 7 | D3 | "return `INVALID_RULE` on bad input" | `mapError` in `errors.go:18` returns `BAD_RULE` | code emits `BAD_RULE`, decision specified `INVALID_RULE` | wrong behaviour |
| 12 | D5 | "audit log on delete" | `deleteSegment` in `service.go:88` — no audit call | required side effect missing | wrong behaviour |
| 6 | D2 | "validate email format" | `validateEmail` in `validator.go:61` | no visible test for the invalid-format path | missing test |

**Not yet started** (items still `[ ]` after this run):

| Item | Spec'd as | D{n} (if known) |
|---|---|---|
| 4 | `validateConditionList` in `validator.go` | D2 |
| 11 | `auth.CheckScope` in `scope.go` | D6 |

### Phase 2 — Design alignment

Bidirectional cross-check. Structural consequences count as tracing to a `D{n}` (route wiring, DI/registry registration, package init, dependency-graph plumbing).

| Direction | For each | Pass | Failure mode | Output |
|---|---|---|---|---|
| Design → Impl | every `D{n}` | at least one impl item traces to it | no impl item | Missing coverage |
| Impl → Design | every impl item | traces to a `D{n}` or structural consequence | neither | Smuggled scope |

This phase produces the alignment table:

| Decision | Impl coverage | Notes |
|---|---|---|
| D1 (field naming) | items 3, 7 | covered |
| D2 (validation placement) | items 5, 6 | covered |
| D3 (error mapping) | item 7 (drifted — see Phase 1) | drift |
| D4 (auth scope) | (no impl item) | **missing — gap** |

| Impl item | Traces to | Notes |
|---|---|---|
| 1 | D1 | ok |
| 9 | (none) | **smuggled — flag user** |

### Phase 3 — Verify

Before producing the report, run these checks. Evidence required — "looks right" is not sufficient.

1. **Impl doc has the on-doc execution rules.** Impl doc contains exactly one `## How to execute this doc` section. If absent, the doc was authored before this skill family adopted the section — flag in the report and offer to add.
2. **No `[ ]` items have `Live reference` populated.** A `Live reference` filled while the box is unticked is a contradiction — either the work is verified (tick it) or the cell is wrong (clear it).
3. **No `[x]` items have empty `Live reference`.** If a box is ticked, the cell must contain a locator per Phase 1's outcome rule: `<symbol> in <file>:<line>` for code, `<file>:<line-range>` for a config/migration block, or `<file>` for a whole-file artifact. If Phase 1 ticked it without filling the reference, the tick is invalid — revert to `[~]` or `[ ]`.
4. **Summary rows resolve against checklist state.** For each row in `## Summary — what changes where`, probe the named file/symbol/artifact-section.

   | Summary row state | Targeting checklist items | Action |
   |---|---|---|
   | File/symbol present | any | pass |
   | Absent | any `[x]` or `[~]` | flag (stale or rolled-back) |
   | Absent | all `[ ]` | no flag (planned, not yet materialized) |
   | Absent | none | flag (smuggled Summary row) |
5. **Sources block is intact.** Sources block exists and each listed path resolves to an existing `RESOLVED` design doc.
6. **No silent overrides from Phase 1.** Every drift row Phase 1 produced must correspond to `[~]` or `[ ]` in the checklist, never `[x]`. If you ticked despite drift, revert.

If any check fails, fix and re-run.

### Phase 4 — Report and propose status

Produce the report in chat (not in the doc — the doc is updated mechanically; the report is the synthesis). Use this exact shape:

```markdown
## Design-task check report

**Impl doc:** `<path>`
**Sources:** `<design-doc-1>`, `<design-doc-2>`
**Run:** <e.g., "after impl-doc P2", "final review">

### Progress
<X of Y items verified, Z partial, W not started> — delta from previous run if applicable (e.g., +3 verified, +1 drifted)

### Newly verified this run
<items flipped to `[x]` — the wins>

| Item | D{n} | Live reference |
| ... | ... | ... |

### Drifted
<items now `[~]` — action items; Category distinguishes "wrong behaviour" (fix code) from "missing test" (add test)>

| Item | D{n} | Decision (verbatim) | Code (file:line) | Divergence | Category |
| ... | ... | ... | ... | ... | ... |

### Not yet started
<items still `[ ]` — visible scope outstanding>

| Item | Spec'd as | D{n} (if known) |
| ... | ... | ... |

### Missing design coverage
<D{n} resolved in design but no impl item traces to it — gap in the impl plan>

| Decision | Notes |
| ... | ... |

### Smuggled scope
<impl items that don't trace to any D{n} — either design is missing the decision, or scope crept into the impl doc>

| Impl item | Notes |
| ... | ... |

### Stale Summary rows
<files/symbols named in `## Summary — what changes where` that don't exist in code — surfaced by Phase 3 sanity check>

| Summary row | Notes |
| ... | ... |

### Status proposal
<one of: NO_CHANGE | IN PROGRESS — phase N | PARTIAL — phases X shipped, Y pending | SHIPPED>

<one-paragraph rationale tying the proposal to the tables above>

### Next steps for the user
1. <concrete follow-up>
2. <concrete follow-up>
```

Omit any section whose table is empty. Don't print empty headers — empty sections are worse than missing ones. If `Newly verified this run` is empty (no items flipped this run), say so explicitly — the user needs to know whether the check made progress.

**Status-flip rules:**
- All boxes `[x]`, no drift, no missing coverage, no smuggled scope → propose `SHIPPED`. User confirms before flip.
- Some boxes `[x]`, rest `[ ]`, no drift → propose `IN PROGRESS — phase N` (highest started phase).
- Mix of `[x]` and `[~]`/`[ ]`, with drift or smuggled scope → propose `IN PROGRESS — phase N (M drifted items)`. Status reflects forward motion; the Drifted table surfaces the action items. Reserve `NO_CHANGE` for cases where execution should actively halt: design-doc gap, unresolved `D{n}`, or user explicitly said "stop until X".
- Mid-execution invocations: usually `IN PROGRESS — phase N`. Update the doc; report stays informational.

Do not flip the status header without an explicit user "yes". Verification is gating, but status changes are user-owned.

---

## Red Flags

Stop and self-correct if you notice any of these mid-execution:

- About to tick `[x]` without having read the code body (Phase 1 step 2).
- Drift table growing past ~5 rows in a single run — halt and surface to the user before continuing.
- About to edit production code — this skill is read-mostly on code.
- Re-verifying an `[x]` item the user didn't explicitly ask to re-check.
- About to mark a `[ ]` item as drift because the code is absent — absence is "not started," not a failure.
- About to flip the impl-doc status header without an explicit user "yes".

---

## Anti-rationalizations

These are pre-written rebuttals to plausible-sounding excuses you'll reach for mid-flow. Read them at the start of every phase. They're written in your future voice — the lies you haven't told yet.

| Excuse | Rebuttal |
|---|---|
| "Most items look done from a quick glance — I'll mark them all `[x]`." | `[x]` requires running the 4-step procedure: identify decision, read code body, verify behaviour, verify tests. "Glance" is not the procedure. |
| "I'll re-verify already-`[x]` items just to be safe." | Don't, unless the user explicitly asked. The skill is idempotent on purpose — re-verifying every run wastes context and time. Trust the previous run's verdict. The user can request a full re-verify with explicit wording. |
| "The design doc didn't quite match what got built, but the build is fine — I'll let it slide." | No. Drift is reported. The user decides whether the design or the code wins. Silent absorption corrupts both. |
| "Tests pass, ship it." | Passing tests are evidence, not proof. Did you check the live behaviour matches the design's `### Decision`? Did you scan for smuggled scope? "Tests pass" is necessary, not sufficient. |
| "I'll modify the code to match the doc." | Out of scope. This skill is read-mostly. Surface the drift; fixing is a separate session. |
| "The named symbol exists — that's enough to tick `[x]`." | No. Existence is step 2 of 4. Steps 3 (behaviour) and 4 (tests) must also pass. "Symbol exists" alone is the failure mode this skill exists to prevent. |
| "I'll flip status to SHIPPED — most things are done, the rest is small." | Status flip is user-owned. Propose, don't impose. |
| "The design doc has an open question, but I'll proceed with the check anyway." | Stop. Open questions in the design = unstable spec. Route back to `design-doc` first. |
| "The user said 'just check it over' — I can skip the design alignment cross-check." | The cross-check is the load-bearing one. "Just check it over" is the user's invocation phrase, not a permission to skip phases. |
| "The user trusts my judgment on what's drift vs not." | They trust the workflow. Drift is reported; you don't pre-classify it as "not really drift" before the user sees it. |
| "The symbol name is descriptive enough — I don't need to read the body." | Read the body. The whole point of Phase 1 is catching code that *names* the right thing but *does* the wrong thing. Skipping the read is the rationalisation Phase 1 exists to defeat. |
| "Code is absent for this `[ ]` item — I should flag it as missing or drift." | No. Absent code = "not started." Not a failure. Leave `[ ]`. The user runs the check incrementally; not-yet-started items are normal mid-flow. |
| "Phase 3 verification is overhead — the report looks complete." | Run the six checks anyway. "Looks complete" ships broken reports. |
| "The symbol got renamed during impl — mark as drift since the doc says `validateX`." | No. Internal renames are not drift if behaviour matches. Update `Live reference` to the actual name. The impl doc tracks roles, not literal spellings. Drift is functional divergence or broken observable contracts (error codes, JSON tags, public API names, DB columns) — not internal renames. |
| "Behaviour matches but no test names the exact boundary — mark drift." | No. Add to Drifted with `Category: missing test`. Test checklist items in the impl doc track test coverage as their own `[ ]` items — that's the natural home for test gaps. "Wrong behaviour" and "missing test" require different actions; keep them distinguishable in the Category column. |

---

## Reference

- `../design-task/SKILL.md` — produces the impl doc this skill verifies.
- `../design-task/template.md` — the impl doc skeleton, including the on-doc `## How to execute this doc` section.
- `../design-doc/SKILL.md` — produces the source-of-truth design docs that the impl doc sources from. Drift detection routes back here when the design itself is the gap.
