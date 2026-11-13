---
name: sendust-security
description: Keep Sendust inside its security envelope — assets and trust boundaries, abuse-case register with evidence owners, TLS/auth policy, input validation, attachment safety, FFI memory safety, redaction rules, DoS caps, and security review checkpoints. Use when implementing SMTP/TLS, attachments, logging, errors, or anything touching the threat model.
---

# Sendust security envelope

Authoritative sources in the sendust repository: `docs/discovery/threat-model.md`, `docs/engineering/security-strategy.md`, `docs/engineering/reliability-strategy.md`, `docs/engineering/test-harness-contract.md`, `docs/architecture/error-model.md`, and `docs/discovery/constraint-ledger.md`. These rules are requirements, not preferences.

**See also:** [sendust-architecture](../sendust-architecture/SKILL.md) for the module boundaries these rules guard · [sendust-evidence](../sendust-evidence/SKILL.md) for gate evidence · [sendust-engineering](../sendust-engineering/SKILL.md) for the workflow · [sendust-knowledge-base](../sendust-knowledge-base/SKILL.md) for TLS/crypto references.

## 1. Assets and trust boundaries

| Asset | Security property |
| --- | --- |
| SMTP credentials | confidentiality; never logged or serialized |
| Message content (subject, body, headers, recipients) | confidentiality and integrity |
| Attachments (buffers and local files) | confidentiality, bounded memory, integrity |
| Relay identity (host, port, EHLO, certificate) | authenticity; untrusted until TLS verification succeeds |
| Process resources (tasks, sockets, fds, memory) | bounded use, deterministic shutdown |
| Native boundary | memory safety, panic containment |
| Release artifacts (.node binaries, npm packages) | integrity, provenance, target correctness |

Trust transitions: consumer → wrapper (untrusted shape) → binding (untrusted values across FFI) → core (owned values, still untrusted content) → lettre (validated envelope) → filesystem (only with explicit opt-in, revalidated descriptor) → relay (untrusted network, TLS mandatory) → error/result back out (redacted). The library is not a sandbox against a fully compromised host process.

## 2. Abuse-case register (mitigation and evidence owner)

| ID | Abuse case | Required mitigation | Evidence owner |
| --- | --- | --- | --- |
| AB-01 | MITM / STARTTLS stripping | implicit TLS or required STARTTLS, never opportunistic; hostname + chain verification | P06-002, P11-002 |
| AB-02 | expired/self-signed/wrong-host/incomplete/malformed certificate | fail closed via rustls verifier + negative fixtures | P06-005, P11-002 |
| AB-03 | CRLF/control injection in address, header, filename, EHLO | reject control chars; reserved headers cannot be overridden | P04-002, P07-001, P11-003 |
| AB-04 | oversized recipients/attachments/wire payload | hard caps before DATA; deterministic validation order | P04-003, P08-003 |
| AB-05 | symlink/FIFO/device/directory attachment path | FS opt-in; no-follow open; fstat; regular files only | P08-002, P11-003 |
| AB-06 | file changes or grows during read | open-then-fstat; bounded read + one-byte overflow probe | P08-002 |
| AB-07 | JS mutates/detaches Buffer after send call | copy bytes synchronously before async work is scheduled | P05-002, P11-004 |
| AB-08 | relay stalls at any stage | outer connect budget, per-operation budget, absolute total deadline, bounded close | P10-002, P10-004 |
| AB-09 | connection drops after DATA terminator | return `unknown`; no automatic retry; drop without QUIT while DATA state is uncertain | P06-004, P10-001 |
| AB-10 | concurrent send or send/close race | synchronous state reservation; immediate BUSY; close atomically enters closing | P04-004, P10-001 |
| AB-11 | panic crosses Node-API boundary | Result for expected failures; unwind-capable build; boundary catch | P04-001, P05-004, P11-004 |
| AB-12 | credentials/body/path leak via diagnostics | structured redaction; marker scans | P09-003, P11-004 |
| AB-13 | external relay/secret in tests | loopback-only fake relay; network-denial test; secret scan | P03-004, P12-004 |
| AB-14 | platform binary mismatch or compromised artifact | exact target matrix, checksums, SBOM, provenance, clean-install tests | P14-003, P16-002 |

## 3. TLS policy

- Modes: implicit TLS and STARTTLS only. Plaintext is not implemented — no flag, env var, or config can disable verification.
- rustls 0.23, `ring` provider, `webpki-roots` anchors; TLS 1.2 minimum, 1.3 preferred; SNI uses the configured relay host.
- Certificate failures close the connection before AUTH or MAIL. Never call lettre's dangerous bypass methods.
- Private enterprise CAs are not trusted in MVP (webpki-roots limitation) — documented trade-off, not a bug.

## 4. Authentication and credential policy

- PLAIN and LOGIN only, after TLS; other mechanisms rejected at configuration time.
- Credentials are owned Rust strings; never copied into errors, logs, or result details; never persisted by Sendust.
- AUTH failure is terminal for the send; no retry with alternate credentials.

## 5. Input validation rules

- Address local parts: ASCII only → else `VALIDATION_NON_ASCII_LOCAL_PART` (SMTPUTF8 not supported in MVP).
- Domains: IDNA conversion; invalid labels rejected.
- Display names and subjects: Unicode allowed, control characters rejected; RFC 2047 encoding when non-ASCII.
- Headers: valid field-name tokens; reserved names cannot be overridden; values reject CRLF, bare LF, NUL, and other control characters.
- Filenames: control characters rejected; RFC 2231/2047 encoding used.
- EHLO identity: hostname token, no CRLF.
- Options: unknown keys rejected (`VALIDATION_INVALID_OPTION`); numeric ranges checked; no silent truncation; limits cannot be raised above hard caps.

## 6. Attachment security

- Filesystem access disabled unless `attachments.allowFileAccess` is explicitly set → else `CONFIG_FILE_ACCESS_DISABLED`.
- Only regular files: no-follow open where supported, then `fstat` the descriptor; reject symlink, directory, FIFO, device, socket, unreadable.
- Read at most cap + 1 byte to detect growth and overflow.
- Absolute paths are redacted from public errors (basename or `<redacted>`); use the caller-supplied filename.
- No URL, stream, glob, or directory attachment form. Buffer attachments are copied before async work.

## 7. Memory, FFI, and panic policy

- No borrowed JS memory crosses an `await`; every buffer/string is copied.
- `unsafe` blocks minimized, justified, and audited.
- Expected failures use `Result`; panics are reserved for violated internal invariants.
- `catch_unwind` at exported napi boundaries; `panic = "abort"` is not used for the addon; a caught panic becomes `INTERNAL` with a generic message.
- Panics on detached Tokio tasks must not leave a transport permanently BUSY — the state machine releases the slot.

## 8. Redaction rules (verified by marker scans)

| Data | Error message | details | Logs |
| --- | --- | --- | --- |
| Password / token | never | never | never |
| Full body | never | never | never |
| Attachment bytes | never | never | never |
| Absolute file path | never | never | never — redact to basename or `<redacted>` |
| Username | redacted/omitted | omitted | omitted |
| Host/port | allowed | allowed | allowed |
| SMTP response code and server text | allowed if credential-free | allowed | allowed |
| Caller-supplied attachment filename | allowed | allowed | allowed |

A redaction regression is a security gate failure (P09-003, P11-004).

## 9. Denial-of-service bounds (hard caps)

100 recipients (`LIMIT_RECIPIENTS`) · 10 MiB per attachment (`LIMIT_ATTACHMENT_BYTES`) · 20 MiB aggregate raw attachments (`LIMIT_ATTACHMENT_TOTAL`) · 30 MiB wire (`LIMIT_WIRE_BYTES`) · timeouts connect 10 s / operation 30 s / total 120 s / close 125 s (`TIMEOUT_*`) · one active send per transport, immediate BUSY · one connection per send, dropped after the transaction, nothing retained idle · bounded close that cannot hang · no unbounded buffering. Queue/job orchestration is caller-owned.

## 10. TLS test fixtures

| Fixture | Expected |
| --- | --- |
| self-signed / expired / wrong host / incomplete chain | `TLS_CERTIFICATE` |
| malformed certificate | `TLS_HANDSHAKE` or `TLS_CERTIFICATE` |
| TLS 1.1 only | `TLS_PROTOCOL` |
| valid test CA | success |

Fixtures are generated by `scripts/generate-tls-fixtures.sh`, committed under `tests/fixtures/tls/`, test-only; no production trust store is modified. All SMTP/TLS tests run on loopback ephemeral ports; no test contacts an external relay or uses real credentials.

## 11. Security review checkpoints

P01 TLS feature validation · P03 dependency/license policy, secret scanning · P05 ownership and panic safety · P06 TLS/auth behavior, fault matrix · P08 filesystem safety and caps · P09 error redaction and public surface · P11 full threat-model audit · P14 artifact and install integrity · P16 SBOM, provenance, secret scan · P17 final security sign-off before publication. A security defect blocks release and is never waived; a released defect means fixed version + deprecation + documented issue, never replaced binaries under the same version.
