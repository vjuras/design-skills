# design-task-check

Verify an implementation tracker doc against actual code and the upstream design doc(s). The **only** thing that ticks boxes — `[x]` means "verified against spec," not "engineer thinks it's done." Idempotent across runs: skips already-`[x]`'d items, so the natural cadence is to invoke iteratively after each impl-doc phase.

## Pipeline

| Step | Actor | Description |
|------|-------|-------------|
| *feature / task* | Human | discovery, feature request, bug — normal team flow; human decides what to build |
| `/design-doc` | AI | analyze codebase; surface and draft decision options |
| resolve | Human + AI | discuss D{n} options; pick one per decision |
| `/design-cleanup` | AI | collapse working draft to slim final form — optional |
| `/design-task` | AI | translate decisions into phased, dependency-ordered impl tracker |
| `/write-code` | AI | implement following on-doc execution rules |
| **`/design-task-check`** | **AI** | **verify each phase against spec; tick `[x]` on match, `[~]` on drift** |
| `/create-pr` | AI | open pull request |
| review | Human | review and approve; AI tools (e.g. CodeRabbit) assist |

## What this skill does

- Reads each `[ ]` or `[~]` item; skips `[x]` items (already verified — trust previous runs).
- Per item, runs the 4-step procedure: identify source `D{n}` → read the artifact's content (code body, migration SQL, config/alert YAML, runbook section) → verify behaviour matches the decision (rule / shape / error code / side effect / algorithm / config-content) → verify tests cover the decision (or smoke/runbook check, where tests don't apply).
- Outcome rules: full match → tick `[x]`; drift → mark `[~]` with concrete divergence note; code absent → leave `[ ]` (not started, not a failure).
- Cross-checks design alignment: every `D{n}` has impl coverage; no impl items lack a `D{n}` trace (smuggled scope).
- Flags stale Summary rows (files/symbols named in the impl doc that don't exist in code).
- Produces an incremental report (newly verified / drifted / not yet started / missing coverage / smuggled scope / stale rows).
- Proposes status transitions; user owns the actual flip.
- **Read-mostly on code, write-only on the impl doc.** Never edits production code; if drift is found, surfaces it for a separate fix session.

## Files

- `SKILL.md` — full instructions.
- (No `template.md` — this skill verifies, doesn't author.)

## Trigger phrases

`/design-task-check`, `/check-impl`, "check progress on this impl doc", "verify implementation against design", "review what's been built".

Natural cadence: after every impl-doc phase. Final pass at PR-review time.
