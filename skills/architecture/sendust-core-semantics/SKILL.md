---
name: sendust-core-semantics
description: Preserve approved Sendust runtime semantics — deterministic validation order, transport state machine and BUSY/close, timeout budgets, SMTP orchestration per RCPT/DATA, outcome rules (accepted/rejected/unknown), and stable error codes. Use when implementing or modifying send(), close(), validation, MIME, or error mapping.
---

# Sendust core semantics

Authoritative sources in the sendust repository: `docs/architecture/system-design.md`, `error-model.md`, `data-flow.md`, `public-api.md`, ADR-0005 (recipient partial failure), ADR-0006 (single send lifecycle), ADR-0007 (timeout budgets), ADR-0009 (hard caps), ADR-0013 (error taxonomy). Exact Rust names are implementation details; the approved semantics are fixed and must not drift.

**See also:** [sendust-crate-and-module-boundaries](../sendust-crate-and-module-boundaries/SKILL.md) for where each behavior lives · [sendust-input-validation-and-caps](../../security/sendust-input-validation-and-caps/SKILL.md) for the per-field rules · [sendust-tls-and-transport-security](../../security/sendust-tls-and-transport-security/SKILL.md) for TLS/auth behavior.

## 1. Data and control flow (per send)

1. Caller constructs a transport with relay, TLS, auth, timeout, attachment, and cap options — validation of transport options is synchronous at `createTransport()`.
2. `send(message)` synchronously prepares an owned native operation, snapshots JS-owned buffers/strings, and reserves the single-send slot; synchronous preparation failures become Promise rejections.
3. The binding schedules the owned operation on napi-managed Tokio 1.
4. Core validates in deterministic order, composes MIME, enforces caps, and opens one `AsyncSmtpConnection` through lettre.
5. lettre performs connect, TLS/required STARTTLS, AUTH, MAIL, one RCPT command per recipient, and one DATA transaction for the accepted-recipient subset.
6. Core maps the final observable state to `accepted`, `rejected`, or `unknown` and closes the session.
7. The binding converts the result or error into a stable JS object; the wrapper returns it unchanged.

No queue, worker, scheduler, queue adapter, or persistence layer exists or is planned. No automatic retry exists at any layer. No per-recipient message fan-out exists.

## 2. Deterministic validation order (first failure wins)

1. Transport options: relay host/port, security mode, auth mode, timeout ranges and relationships (connect/operation ≤ total ≤ close).
2. Message identity: From present, at least one To/CC/BCC.
3. Recipient count and address parsing (ASCII local part, IDNA domain).
4. Header names/values and CRLF/control-character rejection.
5. Subject and body encoding feasibility.
6. Attachment metadata: count, filename, explicit content type validity.
7. Attachment raw bytes caps (buffer attachments first, file attachments after safe read).
8. MIME composition and wire-size cap.
9. Network preconditions (host resolves, config coherent).

A failure at any step returns one error with a stable code. No partial send begins before all validation and composition complete. The order prevents nondeterministic messages and makes tests stable.

## 3. Transport state machine

```text
        close()                drain complete
  open ─────────► closing ─────────────────► closed
    │                │
    │ send()         │ send() -> CLOSED
    ▼                ▼
  sending          (no new send)
    │
    └── outcome -> open
```

- One `send` in flight. A second `send` returns `BUSY` immediately without dialing.
- `close()` is idempotent; concurrent `close()` calls share the same drain.
- `close()` atomically transitions to `closing`, signals cancellation, and waits up to the close budget. Before message transmission begins, cancellation returns a closed/cancelled error or a definitive SMTP rejection. Once `message()` begins, cancellation or transport failure is conservatively `Unknown` unless an explicit final negative SMTP reply is received; drop the connection without issuing QUIT while DATA state is uncertain.
- The close budget (125 s) exceeds the total send budget (120 s) so a close during a maximal send can still drain.
- After `closed`, every `send` returns `CLOSED`.
- No finalizer, GC hook, or process-exit handler is required for correctness.

## 4. Timeout model (hard maximums — may be lowered, never raised)

| Budget | Default | Bounds |
| --- | --- | --- |
| connect | 10 s | DNS + TCP + greeting + EHLO + TLS/STARTTLS negotiation, one outer deadline |
| operation | 30 s | each SMTP command/response wait |
| total | 120 s | the entire send including composition wait and DATA |
| close | 125 s | the bounded close drain |

Single absolute deadline per send: `start + min(total, remaining close budget)`. lettre's inner timeout only bounds TCP attempts — Sendust must wrap the complete connection future. Report only coarse stage boundaries (connection setup; AUTH; MAIL; each RCPT; DATA command; DATA body+terminator+final response; cleanup); never claim internal socket sub-stage attribution. Values are integers ≥ 1 s; zero, negative, sub-second, NaN, infinity, and inconsistent combinations are rejected (`VALIDATION_INVALID_TIMEOUT`).

## 5. SMTP orchestration rules

- One `AsyncSmtpConnection` per send (no pool), created with lettre's public lower-level async API — required for observing each RCPT reply individually. Sendust does not parse SMTP or format commands itself.
- lettre features pinned with `default-features = false` plus `smtp-transport`, `tokio1-rustls`, `ring`, `webpki-roots` (excludes pool and native-tls).
- AUTH: lettre `Credentials`, only `Mechanism::Plain`/`Mechanism::Login`, after TLS; require response 235 — arbitrary 2xx/3xx is not authenticated.
- One `MAIL FROM`; one `RCPT TO` per recipient via `AsyncSmtpConnection::command`; record exact 2xx acceptance and 4xx/5xx rejection via `Error::status()` — lettre treats both 2xx and 3xx as generic positive, so check classes explicitly.
- Continue after ordinary RCPT refusals; RCPT `421` closes the session and stops the envelope. If no recipient is accepted or 421 occurs, do not issue DATA.
- Require DATA command reply 354 before body bytes. Positive final DATA reply (2xx) → overall `accepted`; RCPT-accepted recipients become `accepted`, explicit RCPT refusals stay `rejected`. Explicit negative final DATA reply (4xx/5xx) → overall `rejected`, including RCPT-accepted recipients.
- A non-final/unexpected 3xx after transmission, or a transport error once `message()` begins, yields `unknown` — lettre exposes no public checkpoint proving final disposition. No retry.
- Connection, TLS, and AUTH failures before message submission are operational errors, never `rejected` delivery outcomes.

## 6. MIME structure (deterministic)

- text only → `text/plain`; html only → `text/html`; both → `multipart/alternative`; attachments present → `multipart/mixed` wrapping the body part.
- BCC goes to the SMTP envelope, never into headers. CRLF line endings; legal header folding. Display names and subjects RFC 2047-encoded when non-ASCII; bodies quoted-printable or base64 by a deterministic, tested policy. Wire size is accounted on the final serialized bytes before DATA.

## 7. Outcome and error semantics

- `SendResult.status` ∈ `accepted` | `rejected` | `unknown` — **relay acceptance, never mailbox delivery**. No `delivered`, `queued`, `opened`, or `bounced` status exists in MVP.
- `unknown` is terminal for the call; the caller decides idempotency. `accepted` means the relay accepted responsibility.
- Errors are `SendustError` subclasses with stable `code`, `category`, optional `details`; message strings are never a contract. Codes are append-only within a major version. Full catalog: `docs/architecture/error-model.md` (`VALIDATION_*`, `CONFIG_*`, `LIMIT_*`, `TLS_*`, `AUTH_*`, `PROTOCOL_*`, `TIMEOUT_*`, `TRANSPORT_CLOSED`, `TRANSPORT_BUSY`, `INTERNAL`).
- Panics are reserved for violated internal invariants; caught at exported boundaries as `INTERNAL`; must never leave a transport permanently BUSY.
