---
name: write-code
description: Universal coding-hygiene reminders for any code-writing task. Mirrors the existing code's style instead of imposing one. Reinforces stop-and-ask discipline (don't fabricate stubs/mocks when info is missing), instrumentation at backend I/O boundaries, and comment hygiene (no narrating comments, no design-doc refs in code). Use when implementing features, fixing bugs, refactoring, adding tests, or modifying any code.
---

# write-code

## 1. Match the project as it's written today

Read the file you're editing and nearby files first — naming, error handling, file/folder layout, test patterns. Mirror them. A style refactor is a separate task; don't bundle it.

## 2. Stop and ask — don't fabricate

Don't reinvent stuff like stubs or mocks. If you don't have enough info to complete the task, stop and ask for that info.

Example: task says "implement gRPC client" but you don't have details on where to find it — you have the interface and contract but not the host/credentials. Don't create a stub. Ask.

Triggers to stop and ask:
- Missing endpoint or host
- Missing credentials path
- Missing contract details
- Missing field semantics or business rules

## 3. Instrument at backend I/O boundaries

When adding new backend integration points:
- New external clients (HTTP, gRPC, third-party SDKs, message bus) → wrap in the project's monitoring/metrics decorator
- New storage layers (DB, cache, queue) → same

If the project has no such decorator pattern → match existing observability conventions; don't invent one.

## 4. Comment hygiene

- **Don't add narrating comments**: if you're about to write `// does X`, name the method `doX` instead. Comments are for non-obvious *why*, not restating *what*.
- **No design-doc refs in code**: never cite task numbers, plan sections, spec paragraphs, or checklist items (`// Task-15`, `// §3a`, `// see plan`). Code is not a scratchpad.
