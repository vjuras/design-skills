---
name: design-work-loop
description: Experimental orchestrator that loops `/write-code` → `/design-task-check` over an impl doc's phases until every checklist item is verified `[x]` or a stop condition surfaces. Use when the user says "/design-work-loop", "/work-loop", "execute the impl doc end-to-end", "loop through the phases until done", "auto-implement the impl doc", or has a phased impl tracker (produced by `/design-task`) and wants the implement-then-verify cycle automated phase-by-phase rather than driving each `/write-code` and `/design-task-check` invocation manually.
---

# Design Work Loop — Phase-by-Phase Implement-and-Verify Cycle

> Experimental. Wraps `/write-code` + `/design-task-check` into a per-phase loop. Halts on drift, stop-and-ask events, design forks, or blockers — never silently absorbs them.

This skill does **not** write code or tick boxes itself. It orchestrates the two sub-skills that do. The loop keeps the impl doc honest: code is written → check verifies → next phase begins, with hard stop conditions in between.

## When to use

Trigger phrases: `/design-work-loop`, `/work-loop`, "execute the impl doc end-to-end", "loop through the phases until done", "auto-implement the impl doc".

Pre-conditions:
- An impl doc exists (produced by `/design-task`) with `### Phase N` headings and `[ ]` checklist items.
- Every source design doc listed in the impl doc's `Sources:` block is `RESOLVED` — no `D{n} — PROPOSED` remaining.
- The user has explicitly opted into auto-looping. This is **experimental**: human-in-the-loop review between phases is the default; this skill replaces it with hard stop conditions instead. Confirm before starting.

**When NOT to use:**
- Any source design doc is `DRAFT` or contains `— PROPOSED` decisions — run `/design-doc` first.
- Impl doc has no phases (one flat checklist) — run `/write-code` and `/design-task-check` directly; there's nothing to loop over.
- User wants step-by-step control or to review between phases — drive `/write-code` and `/design-task-check` manually.
- One or two checklist items remain — looping adds overhead for no gain; just run the two sub-skills directly.

## Core principles (do not skip)

1. **This skill orchestrates; it does not write code or tick boxes.** All code edits happen inside `/write-code`. All `[x]` and `[~]` marks happen inside `/design-task-check`. This loop only sequences invocations.
2. **The check is the gate, not the writer.** A phase is not "done" because `/write-code` returned. A phase is done when `/design-task-check` ticks every item in that phase to `[x]` with no drift.
3. **Halt loudly on drift.** `[~]` rows are not auto-fixable here. Surface the report, stop the loop, hand back to the user.
4. **Halt loudly on missing info.** `/write-code` is designed to stop and ask when context is missing (no fabricated stubs, no mocks). When it stops, this loop stops too — never auto-answer in its place.
5. **Status flips are user-owned.** Even after every phase passes, status-header changes (e.g. `IN PROGRESS` → `SHIPPED`) require explicit user "yes."
6. **The doc is the audit trail.** Don't edit the impl doc from this skill. Let `/design-task-check` update it. Don't add notes, follow-ups, or progress markers to the impl doc body.

---

## Workflow

### Phase 0 — Locate and validate

1. Identify the impl doc path. If invoked without one, ask, or scan the current branch's most-recently-modified `*-impl.md` / `*-task.md` and propose.
2. Read the impl doc. Extract its `Sources:` block; confirm each source design doc is `RESOLVED`. If any is `DRAFT` or contains `— PROPOSED`, halt — route the user to `/design-doc`.
3. Extract every `### Phase N` section. Build a phase queue in order. Skip phases where every item is already `[x]` (already shipped or already verified).
4. Announce the plan in chat: phase queue, first phase to be worked, expected stop conditions. Wait for explicit user "go" before entering Phase 1. The skill is experimental — the confirm step is load-bearing.

### Phase 1 — Per-phase loop

For each phase in the queue:

1. **Announce.** "Starting Phase N: <name>. Items: <count>." in chat.
2. **Invoke `/write-code`** scoped to the current phase's items only. Follow `/write-code`'s rules verbatim (mirror existing style, stop-and-ask discipline, instrument I/O boundaries, comment hygiene).
3. **Wait for `/write-code` to return.** Inspect its outcome:
   - Completed cleanly → continue to step 4.
   - Stopped to ask (missing endpoint, credentials, contract, business rule) → **halt.** Surface its question to the user; do not guess.
   - Hit a design fork (item references a decision the design doc didn't resolve) → **halt.** Route user to `/design-doc`.
   - Hit unexpected complexity or a blocker → **halt.** Surface to user.
4. **Invoke `/design-task-check`** with its default (incremental) behaviour — only this phase's `[ ]` / `[~]` items will be re-verified.
5. **Read the check report.** Decide:

| Check outcome | Action |
|---|---|
| All current-phase items now `[x]`, no drift, no smuggled scope | Advance to next phase. |
| Any current-phase item `[~]` (drift) | **Halt.** Surface the drift table to user. Do not enter the next phase. Do not attempt to auto-fix. |
| Any current-phase item still `[ ]` after `/write-code` returned cleanly | **Halt.** Either `/write-code` missed an item, or the item was silently deferred. Surface to user. |
| Check reports missing design coverage or smuggled scope | **Halt.** Surface to user; route back to `/design-doc` or `/design-task` as appropriate. |
| Check itself halted (design-doc gap, sources unresolved, stale Summary rows) | **Halt.** Surface the check's own stop reason. |

6. **If advancing**, log a one-line progress note in chat: "Phase N verified ([x] count). Moving to Phase N+1." Do not edit the impl doc.

### Phase 2 — Wrap-up

When the queue is drained (every phase advanced past, **or** the loop halted):

1. **Success path — every phase passed:**
   - Re-run `/design-task-check` once more as a full-doc sanity pass (it will be a no-op if every item is already `[x]`, but it catches stale Summary rows and missing `Live reference` cells).
   - Confirm Phase 3 verification (see below).
   - Report final state in chat: total items verified, phases completed, anything the check still flagged.
   - Propose the status-header flip (e.g. `IN PROGRESS` → `SHIPPED`). **Wait for explicit user "yes" before flipping.** Status is user-owned.
   - Hand off: "Doc is ready for `/create-pr`."
2. **Halt path — loop stopped mid-phase:**
   - Report which phase, which item, which sub-skill halted, and the reason verbatim.
   - Do not flip status. Do not advance.
   - Hand control back; the user decides next move (fix and resume, change scope, escalate to `/design-doc`).

### Phase 3 — Verify before declaring done

Only on the success path. Evidence required — "looks right" is not sufficient.

1. Zero `[ ]` and zero `[~]` in the impl doc.
2. Every `[x]` item has a populated `Live reference`.
3. `## Summary — what changes where` rows resolve against the codebase (no stale entries).
4. Sources block intact; every source design doc still `RESOLVED`.

If any check fails, halt and surface. Do not paper over.

---

## Red Flags

Stop and self-correct if you notice any of these mid-execution:

- About to auto-answer `/write-code`'s stop-and-ask without user input.
- About to enter the next phase while drift `[~]` rows remain in the current one.
- About to flip the status header without an explicit user "yes."
- About to skip `/design-task-check` between phases because "the write looked clean."
- About to edit the impl doc directly (only `/design-task-check` writes to it).
- About to silently retry a halted phase ("maybe `/write-code` stopped by accident").
- Loop has advanced past 3 phases without surfacing progress — pause and report.

---

## Anti-rationalizations

| Excuse | Rebuttal |
|---|---|
| "`/write-code` returned without errors, so the phase is done — I'll skip the check." | No. The check is the gate. `/write-code` returning is necessary, not sufficient. |
| "Drift is small (one rename, one error code), I'll fix it inline before the next phase." | No. Drift is reported, not absorbed. Halt. The user decides whether to fix code or amend the design. |
| "`/write-code` stopped to ask about credentials — I'll fill in a placeholder and continue." | No. Stop-and-ask is load-bearing. Halt. Surface to user. |
| "Every phase passed, I'll flip status to `SHIPPED` to save the user a step." | Status is user-owned. Propose; never impose. |
| "The impl doc needs a follow-up note — I'll add it inline." | No. The impl doc is `/design-task-check`'s write surface. Surface the follow-up in chat. |
| "I'll re-run `/design-task-check` from scratch each phase to be safe." | No. The check is idempotent by design — `[x]` items are skipped. Default invocation is correct; full re-verify is only on explicit user request. |
| "The user said 'just run the loop' — I can skip Phase 0 validation." | No. Phase 0 validates the inputs are loop-ready. Bypassing it is the most common path to compounding drift. |
| "Phase 2's items look similar to Phase 1's — I'll batch them into one `/write-code` invocation." | No. Phase boundaries exist for a reason (dependency ordering, verification cadence). One phase at a time. |
| "Verification gates are overhead — the user wants speed." | The skill is experimental for exactly this reason. Speed without gates is what the loop replaces — with stricter stop conditions, not looser ones. |
| "Drift is purely 'missing test' category — I'll let the next phase pick it up." | No. Test-coverage drift still means the current phase isn't `[x]`. Halt; let the user decide whether to add the test now or accept the gap. |

---

## Reference

- `../write-code/SKILL.md` — the implementation sub-skill this loop invokes.
- `../design-task-check/SKILL.md` — the verification sub-skill this loop invokes.
- `../design-task/SKILL.md` — produces the phased impl doc this loop consumes.
- `../design-doc/SKILL.md` — drift / design-fork escapes route back here.

## Cross-tool compatibility

This skill follows the Agent Skills standard (frontmatter `name` + `description`, markdown body, progressive disclosure). It works in Claude Code (`~/.claude/skills/`), Cursor (`~/.cursor/skills/`), and any other agent that loads `SKILL.md` files by description-match. The frontmatter `description` above is the load-bearing line for auto-invocation; the body loads only when the agent decides the loop is relevant. No Claude-specific frontmatter fields (`context: fork`, `disable-model-invocation`, `allowed-tools`) are used, so the skill is portable as-is.
