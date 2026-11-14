---
name: sendust-smtp-client-protocol
description: Implement the native SMTP client inside `transports::smtp` — RFC 5321 session state, bounded command/reply parsing, STARTTLS upgrade, AUTH (PLAIN/LOGIN), per-RCPT envelope handling, dot-stuffed DATA, timeout mapping, and reply classification. Use when implementing or testing the SMTP transport.
---

# Sendust native SMTP client protocol

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Authoritative sources in the sendust repository: `docs/architecture/system-design.md` §2.6 and ADR-0017 (native SMTP client, superseding ADR-0002). Protocol authority: RFC 5321 (SMTP), RFC 6409 (submission), RFC 4954 (AUTH), RFC 3207 (STARTTLS), RFC 2920 (PIPELINING), RFC 3030 (CHUNKING/BDAT — not used in MVP), RFC 3463 (enhanced status codes), RFC 1870 (SIZE). Reference implementations (lettre, mail-send) are study material only — not dependencies.

**See also:** [sendust-crate-and-module-boundaries](../sendust-crate-and-module-boundaries/SKILL.md) for where the client lives · [sendust-core-semantics](../sendust-core-semantics/SKILL.md) for the orchestration contract around it · [sendust-tls-and-transport-security](../../security/sendust-tls-and-transport-security/SKILL.md) for the TLS envelope · [sendust-input-validation-and-caps](../../security/sendust-input-validation-and-caps/SKILL.md) for caps enforced before connect.

## 1. Session lifecycle

```text
connect (TCP) → [greeting 220] → EHLO → [250 capabilities]
   ├─ security mode = starttls: STARTTLS → [220] → TLS handshake → EHLO again (capabilities reset)
   ├─ auth configured: AUTH PLAIN / AUTH LOGIN → [235]
   ▼
MAIL FROM:<reverse-path> → [250]
   RCPT TO:<forward-path> × N (one command per recipient, record each reply)
   DATA → [354] → message bytes (dot-stuffed, CRLF, terminated by <CRLF>.<CRLF>) → [final reply 2xx | 4xx/5xx]
   QUIT (when final disposition is certain)
   ▼
outcome: accepted | rejected | unknown
```

Rules: one connection per send, no pooling, no keep-alive; the connection is dropped after the transaction.

## 2. Command framing and bounded reply parsing

- Commands are ASCII, CRLF-terminated (RFC 5321 §4.1.1); max 512 octets per command line. Addresses are validated before connect, so command lines stay short by construction.
- Reply grammar (RFC 5321 §4.2): `nnn <SP> text CRLF` for the final line; intermediate lines use `nnn-` and must carry the same code. Parse the numeric code strictly: exactly three digits; the first digit is the class.
- Bounded reader: cap each reply line (e.g. 1024 octets) and the total multiline response (e.g. 64 lines); exceeding either is a protocol error (`PROTOCOL_UNEXPECTED_RESPONSE`) — never allocate unbounded buffers for relay input.
- Reject NUL and bare CR or LF in reply text as protocol anomalies.

## 3. Reply classification

| Class | Meaning | Sendust treatment |
| --- | --- | --- |
| 2xx | positive completion | success for that step |
| 3xx | positive intermediate (354 after DATA; 33x AUTH challenges) | expected only where the RFC prescribes it; any other 3xx is `PROTOCOL_UNEXPECTED_RESPONSE` and is never counted as success |
| 4xx | transient negative | definitive for this call; no retry |
| 5xx | permanent negative | definitive for this call |
| 421 | service shutting down | session terminates immediately; stop the envelope; no DATA |

Enhanced status codes (RFC 3463, e.g. `5.1.1`) are captured into `details.smtpEnhancedCode` when present; they are never treated as an independent success signal.

## 4. EHLO capabilities

- Parse the multiline 250 response into a capability set (keyword + optional parameters; keywords case-insensitive).
- Extensions Sendust understands: `STARTTLS` (RFC 3207), `AUTH` (RFC 4954, with mechanism list), `PIPELINING` (RFC 2920), `SIZE` (RFC 1870 — compare against the wire cap early when advertised), `8BITMIME`.
- Only advertised capabilities may be used: AUTH only with advertised mechanisms (else `AUTH_UNSUPPORTED`); STARTTLS only when advertised — and mandatory when the security mode requires it, where refusal is a hard error, never a plaintext fallback.
- Extensions are optional by spec: behavior is identical when absent, except where the security mode demands STARTTLS.
- MVP does not pipeline: even when `PIPELINING` is advertised, send one command and wait for its reply — deterministic per-stage timeouts and per-RCPT bookkeeping outweigh round-trip savings. Do not send BDAT/CHUNKING.

## 5. STARTTLS procedure (RFC 3207)

- Implicit TLS (`secure = "tls"`): TLS handshake immediately after TCP connect, before reading the greeting.
- Upgrade (`secure = "starttls"`): greeting → EHLO → `STARTTLS` → expect 220 → TLS handshake on the same connection → EHLO again; discard all knowledge gained before TLS (the pre-TLS capability list is untrusted).
- rustls client config uses `webpki-roots` anchors and verifies the relay host (SNI = configured host); TLS 1.2 minimum. Failure closes the connection before AUTH or MAIL.

## 6. AUTH (RFC 4954)

- Only after TLS is established. Mechanisms:
  - `PLAIN`: base64 of `authzid NUL authcid NUL passwd`, sent as one line after `AUTH PLAIN`.
  - `LOGIN`: `AUTH LOGIN` → expect 334 username challenge → send base64 user → expect 334 password challenge → send base64 password.
- Base64 encoding is produced in owned memory and never logged.
- Require 235 for success. 33x lines are challenges only inside the LOGIN flow. 535 → `AUTH_FAILED` (terminal, no retry). Mechanism not advertised → `AUTH_UNSUPPORTED`, decided before connect.

## 7. Envelope

- `MAIL FROM:<reverse-path>` — the validated From address; a 250 is required to proceed (otherwise overall `rejected`, no RCPT issued).
- One `RCPT TO:<forward-path>` per envelope recipient (To+CC+BCC, max 100). Ordinary per-recipient refusals (4xx/5xx) are recorded and the accepted subset continues; `421` closes the session and prevents DATA (overall `rejected`). If zero recipients are accepted, no DATA is issued.
- No per-recipient message fan-out: one message, one DATA transaction to the accepted-recipient envelope.

## 8. DATA transfer

- Expect 354 before sending any body bytes. A 354 after the message was already transmitted, or any 3xx after transmission, is an anomaly.
- Apply dot-stuffing: any line beginning with `.` gains a leading `.` (undone by the relay); terminate with `<CRLF>.<CRLF>`. Wire bytes (including stuffing) must have passed the 30 MiB wire cap before DATA begins.
- Final reply semantics: 2xx → overall `accepted` (RCPT-accepted recipients become `accepted`; explicit RCPT refusals remain `rejected`). 4xx/5xx final reply → overall `rejected` (RCPT-accepted recipients also become `rejected`).
- If the final reply is not received (write/read error, connection reset, timeout, abrupt close) after the terminator may have reached the relay → `unknown`. Drop the connection without a QUIT while DATA state is uncertain. No retry.

## 9. Timeout mapping

| Stage group | Budget | On expiry |
| --- | --- | --- |
| connect (DNS+TCP+greeting+EHLO+implicit TLS/required STARTTLS negotiation) | connect ≤ 10 s | `TIMEOUT_CONNECT` |
| each command reply wait (EHLO/AUTH/MAIL/RCPT/STARTTLS) | operation ≤ 30 s | `TIMEOUT_OPERATION` |
| entire send including DATA body+terminator+final reply | total ≤ 120 s | `TIMEOUT_TOTAL` |
| bounded close drain | close ≤ 125 s | `TIMEOUT_CLOSE` |

The client owns every stage's outer deadline. A timeout after DATA with an uncertain final disposition becomes `unknown` — never an automatic retry trigger.

## 10. Error mapping

| Client condition | Public code |
| --- | --- |
| TCP connect failure / connection refused | `PROTOCOL_CONNECTION_RESET` (invalid host is caught at validation instead) |
| malformed reply, line limit exceeded, unexpected 3xx | `PROTOCOL_UNEXPECTED_RESPONSE` |
| connection reset before DATA submission | `PROTOCOL_CONNECTION_RESET` |
| connection loss after the DATA terminator may have reached the relay | outcome `unknown` (not an error) |
| AUTH refused | `AUTH_FAILED` / `AUTH_UNSUPPORTED` |
| TLS failure | `TLS_CERTIFICATE` / `TLS_HANDSHAKE` / `TLS_PROTOCOL` |
| STARTTLS refused while the mode demands it | `PROTOCOL_REJECTED` (never plaintext fallback) |
| stall beyond budget | `TIMEOUT_*` |

## 11. Testing

- All tests run against the loopback fake relay harness (P03-004): scripted per-stage responses, per-recipient replies (250/550/451/421), stalls per stage, abrupt closes, multiline and malformed replies, exact command-trace assertions. No sleeps, no external network.
- The fault matrix lives in `docs/engineering/testing-strategy.md` §7 of the sendust repository; the native client must pass every row of it.
- Certificate behavior uses the committed TLS fixtures under `tests/fixtures/tls/` (see [sendust-tls-and-transport-security](../../security/sendust-tls-and-transport-security/SKILL.md)).
