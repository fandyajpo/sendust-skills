---
name: sendust-architecture
description: Apply Sendust architecture rules — the two-crate modular monolith, module ownership, call path, state machine, pinned dependencies, and ADR discipline. Use when changing module structure, adding dependencies, touching crate/runtime boundaries, or implementing any core transport/MIME/lifecycle behavior.
---

# Sendust architecture rules

Authoritative sources in the sendust repository: `docs/architecture/overview.md`, `system-design.md`, `public-api.md`, `data-flow.md`, `error-model.md`, ADR-0001..ADR-0016 (`docs/architecture/decisions/`), and `docs/engineering/dependency-policy.md`. Never contradict them in code.

**See also:** [sendust-engineering](../sendust-engineering/SKILL.md) for the task workflow · [sendust-security](../sendust-security/SKILL.md) for TLS, caps, and redaction · [sendust-docs-sync](../sendust-docs-sync/SKILL.md) for the ADR change procedure · [sendust-evidence](../sendust-evidence/SKILL.md) for gate rules.

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

The TypeScript wrapper is deliberately thin: it validates only public shape and option compatibility that generated types cannot express. It MUST NOT reinterpret core statuses or implement SMTP/MIME policy — this prevents two implementations from drifting. The napi adapter isolates all Node-API and unsafe-adjacent code. The core is testable without Node.

## 2. The two-crate boundary

- `crates/sendust-core` — the runtime-independent engine. Must not depend on Node.js, napi, or any JavaScript runtime.
- `crates/sendust-napi` — a thin inbound Node-API adapter. Depends on core; the reverse is forbidden. Contains no MIME or SMTP policy.
- `ts/src` — public JS API and runtime loader only; must not duplicate core validation, MIME, or delivery decisions. Native loading stays isolated in `ts/src/loader.ts`.
- A third crate (`sendust-mime`, `sendust-smtp`, …) is not justified in MVP: MIME and SMTP share the message model and validation policy. Extraction is post-MVP and requires an ADR.

## 3. Modular monolith — module ownership (ADR-0016)

Logical modules inside `sendust-core/src/` (names may flex, ownership must not):

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

Dependency rules enforced by static architecture checks (`scripts/check-architecture.mjs`): domain modules must not import lettre, SMTP session types, napi, Node, or JS runtime APIs. `delivery` depends on the internal port; `transports::smtp` implements it.

## 4. Core semantics every implementation must respect

- **Validation order (deterministic, first failure wins):** transport options → message identity (From present, ≥1 To/CC/BCC) → recipient count and address parsing → header names/values with CRLF/control rejection → subject/body encoding → attachment metadata → attachment byte caps → MIME composition and wire cap → network preconditions.
- **State machine:** `open → sending → open` on outcome; `open/closing → closed` via `close()`. One send in flight; a second returns `BUSY` without dialing. `close()` is idempotent, atomically enters `closing`, signals cancellation, waits up to the close budget (125 s > total send budget 120 s). After `closed`, every `send` returns `CLOSED`. No GC finalizer or process-exit hook is required for correctness.
- **Timeouts (hard maximums, may be lowered not raised):** connect 10 s, operation 30 s, total 120 s, close 125 s. One absolute deadline per send: `start + min(total, remaining close budget)`.
- **SMTP orchestration:** one `AsyncSmtpConnection` per send (no pool), lettre's public lower-level async API for per-RCPT replies. One `MAIL FROM`, one `RCPT TO` per recipient, exactly one DATA transaction for the accepted-recipient subset. Check response classes explicitly (lettre treats 2xx and 3xx alike); require 235 for AUTH; require 354 before body bytes. Mixed envelope: refused recipients stay `rejected`, accepted subset proceeds. RCPT 421 closes the session and prevents DATA. Explicit negative final DATA reply → overall `rejected`; missing final reply after DATA may have reached the relay → `unknown`. No retry anywhere.
- **Ownership model:** JS values are valid only during the synchronous call. Every Buffer/ArrayBuffer/typed array/string is copied into Rust-owned memory before async work is scheduled. In-flight work holds its own owned state and never borrows JS memory. Blocking file reads run on a blocking pool, never the reactor.

## 5. Public API and error semantics (summary)

The TypeScript contract (`public-api.md`): `createTransport(options)` → `Transport` with `send(message): Promise<SendResult>`, `close(): Promise<void>`, `state: "open" | "closing" | "closed"`. `SendResult.status` is `accepted` | `rejected` | `unknown` — **relay acceptance, never mailbox delivery**; no `delivered`/`queued`/`bounced` status exists in MVP. Errors are `SendustError` subclasses with stable `code`, `category`, optional `details`; message strings are never a contract. Codes are append-only within a major version.

## 6. Pinned dependencies (policy)

| Crate | Pin | Features | Notes |
| --- | --- | --- | --- |
| `lettre` | `=0.11.23` | `smtp-transport`, `tokio1-rustls`, `ring`, `webpki-roots`, default off | only MVP transport; builder feature not enabled |
| `napi` / `napi-derive` / `napi-build` | `=3.16.2` / `=3.7.0` / `=2.6.0` | `napi8`, `async` | Node-API 8 binding |
| `tokio` | 1.x | — | runtime managed by napi; never create another |

Toolchain pins (Rust 1.100.0, npm 12.2.2, TypeScript 7.0.2, Biome 2.6.7, Vitest 5.1.0) live only in `docs/engineering/toolchain-baseline.md` — link, never copy. Allowed licenses: Apache-2.0, MIT, BSD-3-Clause, ISC, Unicode-3.0, CDLA-Permissive-2.0; anything else needs a recorded decision. `Cargo.lock` committed; one root `package-lock.json`; no pnpm/yarn/bun lockfiles; no postinstall downloads.

## 7. Adding a dependency

1. State the requirement/constraint ID and why stdlib + existing pins cannot satisfy it.
2. Record license, version, source; check maintenance and advisory history.
3. Confirm it does not cross the core/binding boundary, add runtime-specific behavior to shared code, or duplicate SMTP/MIME/validation policy.
4. Pin exact version with the smallest feature set; verify `cargo tree -e features` and `cargo tree -d`.
5. Update dependency policy, toolchain baseline (if a pin changes), phase plan, and lockfile in the same change.

A dependency without a recorded justification is rejected in review even if it compiles. A dependency that pulls OpenSSL, `native-tls`, or a second TLS stack blocks the change.

## 8. Prohibited in MVP

A custom SMTP stack or parser (lettre is the only implementation) · insecure TLS toggles, plaintext fallback, certificate/hostname bypass · URL/HTTP/remote attachment fetching · connection pooling, automatic retry, internal queues, workers, schedulers, persistence · a third crate, workspace, plugin registry, or alternate transport · Node/napi/JS types inside core · SMTP/MIME/validation/delivery policy inside napi or ts/src · `any` in public TypeScript · telemetry SDKs that could carry message content or credentials.

Also deliberately excluded by the product (do not "helpfully" add): SMTPUTF8, connection keep-alive, per-recipient fan-out, AbortSignal as a required mechanism, URL or streamed attachments, extra transports (SES/SendGrid/etc.), templates, tracking, analytics. Queue/job orchestration is `HELD`, not post-MVP.

## 9. Changing architecture

- Never change an accepted ADR silently. A material change requires a new superseding ADR plus synchronized contract/task/gate updates (sendust-docs-sync).
- Architecture amendment approval is design approval only, never implementation approval.
- Runtime/platform support cannot be claimed until exact runtime × OS × CPU × ABI/libc combinations pass blocking Sendust CI (ADR-0015). Deno requires local `node_modules` and `--allow-ffi`.
