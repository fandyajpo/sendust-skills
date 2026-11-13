---
name: sendust-engineering
description: Implement a Sendust task end-to-end following the AI implementation contract. Use when implementing, planning, or reviewing any Sendust phase task (Pxx-00y) in the sendust repository.
---

# Sendust engineering workflow

Sendust is a Rust native email delivery engine exposing a TypeScript API over a Node-API adapter, targeting Node.js, Bun, and Deno. Work in the sendust repository is doc-driven and task-based; this skill is the workflow. The normative contract is `docs/engineering/ai-implementation-contract.md` and `AGENTS.md` in the sendust repository.

## Required reading before any change

In this order:

1. `README.md`, `docs/README.md`
2. `docs/engineering/toolchain-baseline.md`
3. `docs/product/vision.md`, `requirements.md`, `mvp-scope.md`
4. `docs/discovery/constraint-ledger.md`
5. `docs/architecture/overview.md`, `system-design.md`, `public-api.md`, `data-flow.md`, `error-model.md`
6. Relevant accepted ADRs (index: `docs/architecture/decisions/README.md`)
7. The relevant phase plan in `docs/phases/`
8. The matching task row in `docs/planning/task-registry.md`, dependencies in `docs/planning/dependency-graph.md`, gates in `docs/planning/quality-gates.md`

The phase plan owns task-level detail. Requirements own product behavior. Accepted ADRs own architecture. The task registry and dependency graph index work; they must not silently redefine it. If two authoritative documents conflict, stop the change, report the exact conflict and paths, and do not pick whichever text is convenient.

## Pre-edit verification

- The task exists exactly once in the task registry and the phase plan.
- All prerequisites are complete **with evidence** — never trust a prior agent's completion claim without checking the files.
- The task is not blocked by an unresolved decision.
- The target files do not already implement the behavior; the change does not duplicate or contradict existing code.

## Workflow per task

1. **Inspect** — read the full task and every linked document (not just titles).
2. **Plan** — list files to create/change, module ownership, interfaces, tests, doc updates. Keep it scoped.
3. **Implement** — only the assigned task plus necessary supporting changes. Preserve public contracts and module boundaries. No speculative abstractions, no unrelated refactors.
4. **Static review** — inspect syntax, imports/exports, types and trait bounds, ownership/borrowing/Send/Sync, TypeScript strictness and runtime-neutral assumptions, dependency direction, API parity across Rust/N-API/TypeScript, redaction, caps, cleanup, and whether tests actually assert the requirement. Static review is not compilation.
5. **Synchronize documentation** — see the sendust-docs-sync skill; update dependents in the same change.
6. **Report** — use the structure in `docs/engineering/ai-implementation-contract.md` §9 (task ID, prerequisites, files, implementation, traceability IDs, static review, execution labels, external validation, blockers, next eligible task).

## Capability disclosure

If you cannot execute terminal commands, builds, tests, or linters:

- Do not claim any of them ran or passed.
- Label your work as static review.
- List the exact commands and environment required for external validation.
- Do not mark a task or gate complete when acceptance criteria require executable evidence.

## Hard constraints (non-negotiable)

- Exactly two Rust crates: `crates/sendust-core` (no Node/napi/JS dependency) and `crates/sendust-napi` (thin adapter depending on core, never the reverse).
- TypeScript owns the public facade and the isolated native loader, not SMTP/MIME business policy.
- SMTP via lettre is the only MVP outbound transport; SMTP protocol behavior lives only in `transports::smtp`.
- TLS certificate and hostname verification are mandatory; no insecure bypass or plaintext fallback.
- One active send per transport (second concurrent send fails with BUSY); close is explicit, idempotent, and bounded.
- No placeholders, `todo!`, `unimplemented!`, or fake success paths.
- No `any` in TypeScript public contracts; keep Rust and TypeScript types explicit.
- Never place credentials, message bodies, attachment bytes, or unredacted filesystem paths in logs or errors.
- Do not ask the user to resolve decisions already made in approved documentation; do not invent new architectural decisions. For a genuinely unresolved decision, record the question and propose an ADR.
