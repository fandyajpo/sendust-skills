---
name: sendust-engineering
description: Implement a Sendust task end-to-end following the AI implementation contract — required reading order, authority precedence, workflow, hard constraints, and the phase map. Use when implementing, planning, or reviewing any Sendust phase task (Pxx-00y) in the sendust repository.
---

# Sendust engineering workflow

Sendust is a Rust native email delivery engine exposing a TypeScript API over a Node-API adapter, targeting Node.js, Bun, and Deno. Work in the sendust repository is doc-driven and task-based. The normative contract is `docs/engineering/ai-implementation-contract.md` and `AGENTS.md` in the sendust repository; this skill operationalizes them.

**See also:** [sendust-architecture](../sendust-architecture/SKILL.md) for boundaries and module ownership · [sendust-evidence](../sendust-evidence/SKILL.md) for reporting and validation · [sendust-docs-sync](../sendust-docs-sync/SKILL.md) when any source-of-truth document changes · [sendust-security](../sendust-security/SKILL.md) near the threat boundary · [sendust-knowledge-base](../sendust-knowledge-base/SKILL.md) for protocol and runtime references.

## 1. Required reading before any change

In this order:

1. `README.md`, `docs/README.md`
2. `docs/engineering/toolchain-baseline.md` — single source of truth for every version pin
3. `docs/product/vision.md`, `requirements.md`, `mvp-scope.md`
4. `docs/discovery/constraint-ledger.md` (C-01–C-40), `threat-model.md`
5. `docs/architecture/overview.md`, `system-design.md`, `public-api.md`, `data-flow.md`, `error-model.md`
6. Relevant accepted ADRs — index at `docs/architecture/decisions/README.md`
7. The relevant phase plan in `docs/phases/`
8. The matching task row in `docs/planning/task-registry.md`, dependencies in `dependency-graph.md`, gates in `quality-gates.md`

Ownership rules: the phase plan owns task-level detail. Requirements own product behavior. Accepted ADRs own architecture. The task registry and dependency graph index work; they must not silently redefine it. If two authoritative documents conflict, stop the change, report the exact conflict and both paths, and do not pick whichever text is convenient.

## 2. Authority precedence

1. Tool versions and runtime candidates: `toolchain-baseline.md`; no other document may change a pin.
2. Product behavior and scope: `requirements.md`, then `mvp-scope.md`, `vision.md`.
3. Approved architecture: accepted ADRs; a later superseding ADR beats an earlier one.
4. Architecture contracts: overview, system design, public API, data flow, error model.
5. Implementation detail: the task section in `docs/phases/phase-XX.md`.
6. Execution ordering: task registry and dependency graph.
7. Summaries and navigation: READMEs, phase index, roadmap, matrices, strategies.

A lower-level document may add detail but cannot override a higher-level decision. A conflict between equal or higher authorities is a blocker: identify both passages, do not implement the disputed behavior, propose a tracked correction. Never silently resolve an architecture conflict in code.

## 3. Pre-edit verification

- The task exists exactly once in the task registry and the phase plan.
- All prerequisites are complete **with evidence** — never trust a prior agent's completion claim without checking the files.
- The task is not blocked by an unresolved decision.
- The target files do not already implement the behavior; the change does not duplicate or contradict existing code.

## 4. Workflow per task

1. **Inspect** — read the full task and every linked document (not just titles). Inspect current source, manifests, tests, and history.
2. **Plan** — list files to create/change, module ownership, interfaces, tests, doc updates, risks. Keep the plan scoped to the task.
3. **Implement** — only the assigned task plus necessary supporting changes. Preserve public contracts and module boundaries. No speculative abstractions, no unrelated refactors. Follow the deterministic validation order and module ownership.
4. **Static review** — inspect syntax and delimiters; imports/exports/visibility; type compatibility and trait bounds; Rust ownership, borrowing, lifetimes, Send/Sync, error propagation; TypeScript strictness, generics, Promise contracts, runtime-neutral assumptions; feature flags and dependency direction; API/serialization parity across Rust, N-API, and TypeScript; security boundaries, redaction, caps, timeout semantics, cleanup; whether tests actually assert the requirement; internal links and ID references. Static review is not compilation and cannot prove runtime behavior.
5. **Synchronize documentation** — every affected dependent document in the same change (see sendust-docs-sync).
6. **Report** — use the task report structure in sendust-evidence.

## 5. Capability disclosure

If you cannot execute terminal commands, builds, tests, or linters:

- Do not claim any of them ran, compiled, or passed.
- Label your work as static review only.
- List the exact commands and environment required for external validation.
- Do not mark a task or gate complete when its acceptance criteria require executable evidence.
- You may mark a deliverable "implemented but validation-pending" only where the status model allows it and the distinction is explicit.

## 6. Phase map

P00–P03 are COMPLETED with approved gate evidence; A01 (multi-runtime) and A02 (modular monolith) amendments are approved at design level; P04–P18 not started.

| Phase | Name | Gate task | Status |
| --- | --- | --- | --- |
| P00 | Discovery | P00-005 | COMPLETED |
| P01 | Research | P01-005 | COMPLETED |
| P02 | Architecture | P02-005 | COMPLETED |
| P03 | Repository baseline | P03-005 | COMPLETED |
| P04 | Rust core (errors, validation, message model, state machine, SMTP orchestration) | P04-005 | NOT STARTED |
| P05 | Binding (Node-API 8, owned memory) | P05-005 | NOT STARTED |
| P06 | SMTP (relay sessions, TLS/auth, recipient policy) | P06-005 | NOT STARTED |
| P07 | MIME (headers, alternative bodies, deterministic serialization) | P07-005 | NOT STARTED |
| P08 | Attachments (opt-in files, caps) | P08-005 | NOT STARTED |
| P09 | Public API (TypeScript transport, send, close, errors) | P09-005 | NOT STARTED |
| P10 | Reliability (races, timeouts, cleanup, fault injection) | P10-005 | NOT STARTED |
| P11 | Security (threat-model audit, redaction) | P11-005 | NOT STARTED |
| P12 | QA (traceability, contract tests) | P12-005 | NOT STARTED |
| P13 | Performance (budgets, benchmarks) | P13-005 | NOT STARTED |
| P14 | Distribution (platform matrix, clean installs) | P14-005 | NOT STARTED |
| P15 | Documentation (quick start, lifecycle) | P15-005 | NOT STARTED |
| P16 | Release engineering (versioning, reproducible build) | P16-005 | NOT STARTED |
| P17 | RC and MVP approval | P17-005 | NOT STARTED |
| P18 | Post-MVP | P18-005 | NOT STARTED |

Genuine parallelism: P07 (MIME) can run with P06 (SMTP) after P04; P15 can begin after P09 and run beside P10–P14 (P15-004 needs P11-005); P14-001–P14-003 can start after P05/P03 beside P09–P13. Task IDs Pxx-001–Pxx-005 per phase; they are stable and must not be renumbered.

## 7. Hard constraints (non-negotiable)

- Exactly two Rust crates: `crates/sendust-core` (no Node/napi/JS dependency) and `crates/sendust-napi` (thin adapter depending on core, never the reverse).
- TypeScript owns the public facade and the isolated native loader, not SMTP/MIME business policy.
- SMTP via lettre is the only MVP outbound transport; SMTP protocol behavior lives only in `transports::smtp`.
- TLS certificate and hostname verification are mandatory; no insecure bypass or plaintext fallback.
- One active send per transport (second concurrent send fails with BUSY); close is explicit, idempotent, and bounded.
- No placeholders, `todo!`, `unimplemented!`, or fake success paths; no placeholders reported complete.
- No `any` in TypeScript public contracts; keep Rust and TypeScript types explicit.
- Never place credentials, message bodies, attachment bytes, or unredacted filesystem paths in logs or errors.
- Do not ask the user to resolve decisions already made in approved documentation; do not invent new architectural decisions. For a genuinely unresolved decision, record the question and propose an ADR.
