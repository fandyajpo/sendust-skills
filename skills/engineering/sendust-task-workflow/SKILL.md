---
name: sendust-task-workflow
description: Execute a Sendust repository task (Pxx-00y) correctly — required reading order, authority precedence, pre-edit checks, implementation steps, static review, and capability disclosure. Use when starting, implementing, or reviewing any Sendust task.
---

# Sendust task workflow

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Work in the sendust repository is doc-driven and task-based. The normative contract is `docs/engineering/ai-implementation-contract.md` and `AGENTS.md` in the sendust repository; this skill operationalizes them for a single task.

**See also:** [sendust-phase-map](../sendust-phase-map/SKILL.md) to find the task and its prerequisites · [sendust-crate-and-module-boundaries](../../architecture/sendust-crate-and-module-boundaries/SKILL.md) before moving code · [sendust-evidence-and-gates](../../quality/sendust-evidence-and-gates/SKILL.md) when reporting · [sendust-docs-sync](../../documentation/sendust-docs-sync/SKILL.md) when any source-of-truth document changes.

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
6. **Report** — use the task report template in sendust-evidence-and-gates.

## 5. Capability disclosure

If you cannot execute terminal commands, builds, tests, or linters:

- Do not claim any of them ran, compiled, or passed.
- Label your work as static review only.
- List the exact commands and environment required for external validation.
- Do not mark a task or gate complete when its acceptance criteria require executable evidence.
- You may mark a deliverable "implemented but validation-pending" only where the status model allows it and the distinction is explicit.

## 6. Hard constraints (non-negotiable)

- Exactly two Rust crates: `crates/sendust-core` (no Node/napi/JS dependency) and `crates/sendust-napi` (thin adapter depending on core, never the reverse).
- TypeScript owns the public facade and the isolated native loader, not SMTP/MIME business policy.
- SMTP is delivered by the in-house native client (tokio 1 + rustls 0.23, per ADR-0017 superseding ADR-0002); SMTP protocol behavior lives only in `transports::smtp`.
- TLS certificate and hostname verification are mandatory; no insecure bypass or plaintext fallback.
- One active send per transport (second concurrent send fails with BUSY); close is explicit, idempotent, and bounded.
- No placeholders, `todo!`, `unimplemented!`, or fake success paths reported as complete.
- No `any` in TypeScript public contracts; keep Rust and TypeScript types explicit.
- Never place credentials, message bodies, attachment bytes, or unredacted filesystem paths in logs or errors.
- Do not ask the user to resolve decisions already made in approved documentation; do not invent new architectural decisions. For a genuinely unresolved decision, record the question and propose an ADR.
