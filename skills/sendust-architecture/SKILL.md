---
name: sendust-architecture
description: Apply Sendust architecture rules — two-crate modular monolith, dependency direction, ADR discipline. Use when changing module structure, adding dependencies, or proposing architecture changes.
---

# Sendust architecture rules

Authoritative sources in the sendust repository: `docs/architecture/overview.md`, `system-design.md`, `public-api.md`, `data-flow.md`, `error-model.md`, ADR-0001..ADR-0016 (`docs/architecture/decisions/`), and `docs/engineering/dependency-policy.md`. Never contradict them in code.

## The two-crate boundary

- `crates/sendust-core` — the runtime-independent engine. Must not depend on Node.js, napi, or any JavaScript runtime.
- `crates/sendust-napi` — a thin inbound Node-API adapter. Depends on core; the reverse is forbidden.
- TypeScript owns the public facade and the isolated native loader. Business policy (SMTP, MIME) never moves into TypeScript.
- Node.js, Bun, and Deno are required MVP runtime targets (ADR-0015 / Amendment A01), but support may not be claimed until exact-version, exact-platform blocking CI exists in Sendust-owned CI.

## Modular monolith (ADR-0016 / Amendment A02)

- The core is a modular monolith; module ownership is fixed by ADR-0016 and enforced by `scripts/check-architecture.mjs` in CI.
- SMTP protocol behavior lives only in `transports::smtp`; other modules go through narrow internal delivery contracts.
- Do not add a third crate, a microservice, a plugin registry, an alternate transport, a queue, worker, scheduler, persistence layer, connection pool, automatic retry, URL attachment fetch, or GC-dependent lifecycle behavior. None of these are in MVP scope.

## MVP product boundaries

- SMTP-first: implicit TLS and STARTTLS with mandatory certificate and hostname verification.
- One message per `send()` call; one active send per transport (a second concurrent send fails immediately with BUSY); close is explicit, idempotent, and bounded.
- No automatic retry, pooling, queueing, or background delivery. A successful send means the SMTP relay accepted responsibility, not that a mailbox received the message.
- Queueing, scheduling, and retry orchestration belong to the consuming application.

## Adding a dependency

Before adding any dependency, record: necessity, exact version and feature policy (`docs/engineering/dependency-policy.md`), license, maintenance health, security posture, and impact on the core/runtime boundary. Prohibited features from the dependency policy stay prohibited.

## Changing architecture

- Never change an accepted ADR silently. A material architecture change requires a new superseding ADR plus synchronized contract/task/gate updates (see the sendust-docs-sync skill).
- A lower-level document may add detail but cannot override a higher-level decision. An architecture conflict between equal or higher authorities is a blocker: identify both passages, do not implement the disputed behavior, propose a tracked correction.
- Architecture amendment approval is design approval only, never implementation approval.
