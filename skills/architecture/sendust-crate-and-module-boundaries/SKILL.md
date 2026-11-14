---
name: sendust-crate-and-module-boundaries
description: Apply Sendust structural rules — the two-crate workspace, the TS → napi → core → lettre call path, module ownership inside sendust-core (ADR-0016), and dependency direction. Use when creating or moving code between modules, or touching the TypeScript/native/core boundaries.
---

# Sendust crate and module boundaries

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Authoritative sources in the sendust repository: `docs/architecture/overview.md`, `system-design.md`, ADR-0001 (two crates), ADR-0004 (napi async/tokio), ADR-0008 (owned memory), ADR-0015 (multi-runtime), ADR-0016 (modular monolith), and `scripts/check-architecture.mjs`.

**See also:** [sendust-core-semantics](../sendust-core-semantics/SKILL.md) for runtime behavior rules · [sendust-dependency-policy](../sendust-dependency-policy/SKILL.md) before adding any dependency · [sendust-task-workflow](../../engineering/sendust-task-workflow/SKILL.md) for the process.

## 1. Shape and call path

```text
Consumer application (Node.js / Bun / Deno)
        │  import { createTransport } from "sendust"
        ▼
ts/src            public API, option shape, result/error classes, ESM+CJS entries
        │  calls generated napi exports
        ▼
crates/sendust-napi    Node-API adapter: copy inputs, map errors, own async tasks
        │  calls core types and functions
        ▼
crates/sendust-core    validation, message model, MIME, transport state, SMTP orchestration
        │  uses
        ▼
lettre 0.11.23 (Tokio 1 executor)   SMTP session, TLS, AUTH, DATA
        ▼
Consumer-configured SMTP relay
```

Boundary responsibilities and reasons:

| Boundary | Responsibility | Reason |
| --- | --- | --- |
| TypeScript wrapper | public names, types, defaults documentation, error classes, module formats | small native surface; JS ergonomics change without recompiling Rust |
| napi adapter | convert JS values to owned Rust values; map results/errors; schedule async work | isolates all Node-API and unsafe-adjacent code |
| Rust core | all protocol-independent policy and all SMTP/MIME behavior | testable without Node; enforces validation, limits, lifecycle |
| lettre | SMTP/TLS/AUTH/MIME primitives | avoids maintaining an SMTP client; pinned |

The TypeScript wrapper is deliberately thin. It validates only public shape and option compatibility that generated types cannot express. It MUST NOT reinterpret core statuses or implement SMTP/MIME policy — this prevents two implementations from drifting.

## 2. The two-crate boundary

- `crates/sendust-core` — the runtime-independent engine. Must not depend on Node.js, napi, or any JavaScript runtime.
- `crates/sendust-napi` — a thin inbound Node-API adapter. Depends on core; the reverse is forbidden. Contains no MIME or SMTP policy.
- `ts/src` — public JS API and runtime loader only; must not duplicate core validation, MIME, or delivery decisions. Native loading stays isolated in `ts/src/loader.ts`.
- A third crate (`sendust-mime`, `sendust-smtp`, …) is not justified in MVP: MIME and SMTP share the message model and validation policy. Extraction is post-MVP and requires an ADR.

## 3. Module ownership inside sendust-core (ADR-0016)

Logical modules inside `sendust-core/src/` (folder names may flex, ownership must not):

| Module | Owns |
| --- | --- |
| `lib.rs` | narrow public facade; re-exports only what napi and tests need |
| `config/` | relay, TLS/auth, timeout and limit config |
| `address/` | mailbox parsing and normalization (ASCII local part, IDNA domain) |
| `message/` | message and header model |
| `validation/` | deterministic cross-field and injection checks |
| `attachments/` | bounded buffer/file attachment policy |
| `mime/` | MIME tree, encoding, wire-size accounting |
| `delivery/` | send use-case orchestration and delivery certainty |
| `ports/` | narrow internal outbound delivery contract |
| `transports/smtp/` | lettre session, TLS, AUTH, envelope, RCPT/DATA — the only MVP adapter |
| `lifecycle/` | open/closing/closed, BUSY, bounded close |
| `errors/` and `results/` | internal error taxonomy; accepted/rejected/unknown outcomes |

Dependency direction enforced by static architecture checks (`scripts/check-architecture.mjs`, re-audited at P12):

- Domain modules (address, message, config value types, results, core validation) must not import lettre, SMTP session types, napi, Node, or JS runtime APIs.
- `delivery` depends on the internal port in `ports/`; `transports::smtp` implements that port and is the only MVP adapter.
- `lib.rs` re-exports only what `sendust-napi` and tests need; internal modules are private by default.
- No crate per module, no public plugin system, no runtime-specific business engines, no empty feature hooks, no extra transport implementations.

## 4. Runtime boundary (ADR-0015)

- The public TypeScript API is runtime-neutral; native loading is isolated in `ts/src/loader.ts`.
- Shared TypeScript code must not assume Node-only globals or built-ins.
- A minimal runtime-specific loader branch is permitted; separate Rust adapters or another crate are not, without a failing compatibility test and a superseding ADR.
- Node.js, Bun, and Deno are required MVP targets, but none may be claimed supported until exact runtime × OS × CPU × ABI/libc blocking CI passes. Deno needs local `node_modules` and `--allow-ffi`; no reliance on postinstall scripts or install-time compilation.
