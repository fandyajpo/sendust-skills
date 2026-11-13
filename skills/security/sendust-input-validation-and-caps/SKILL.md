---
name: sendust-input-validation-and-caps
description: Validate every Sendust input and enforce the hard caps — per-field validation rules, header injection rejection, attachment filesystem policy, and DoS bounds. Use when implementing validation, MIME inputs, attachments, limits, or their negative tests.
---

# Sendust input validation and caps

Sources in the sendust repository: `docs/engineering/security-strategy.md` §5–§6 and §9, `docs/architecture/system-design.md` §2.2/§2.7, `docs/architecture/error-model.md`, `docs/engineering/testing-strategy.md` §5. Related abuse cases: AB-03, AB-04, AB-05, AB-06, AB-07 (see [sendust-threat-register](../sendust-threat-register/SKILL.md)).

**See also:** [sendust-core-semantics](../../architecture/sendust-core-semantics/SKILL.md) for the deterministic validation order these rules slot into · [sendust-crate-and-module-boundaries](../../architecture/sendust-crate-and-module-boundaries/SKILL.md) for the owning modules.

## 1. Per-field validation rules

| Field | Rule | Error code on failure |
| --- | --- | --- |
| Transport options | unknown keys rejected; numeric ranges checked; no silent truncation | `VALIDATION_INVALID_OPTION` |
| Timeouts | integers ≥ 1 s; connect/operation ≤ total; close ≥ total; no zero/negative/sub-second/NaN/infinity | `VALIDATION_INVALID_TIMEOUT` |
| From | required | `VALIDATION_MISSING_FROM` |
| Recipients | at least one of To/CC/BCC | `VALIDATION_MISSING_RECIPIENT` |
| Address | parseable; ASCII local part; IDNA-normalized domain | `VALIDATION_INVALID_ADDRESS` / `VALIDATION_NON_ASCII_LOCAL_PART` (SMTPUTF8 not supported in MVP) |
| Headers | valid field-name tokens; reserved names cannot be overridden | `VALIDATION_INVALID_HEADER` |
| Header values | reject CRLF, bare LF, NUL, and other control characters | `VALIDATION_HEADER_INJECTION` |
| Display names / subject / body | Unicode allowed; control characters rejected; RFC 2047 encoding when non-ASCII | `VALIDATION_HEADER_INJECTION` / encoding errors |
| Attachment metadata | filename required; explicit content type valid; supported shape only | `VALIDATION_INVALID_ATTACHMENT` |
| Filenames | control characters rejected; RFC 2231/2047 encoding used | `VALIDATION_INVALID_ATTACHMENT` |
| EHLO identity | hostname token, no CRLF | configuration/validation error |

Limits cannot be raised above the hard caps — a caller-provided larger limit is itself rejected.

## 2. Attachment filesystem policy

- Filesystem access is disabled unless `attachments.allowFileAccess` is explicitly set → `CONFIG_FILE_ACCESS_DISABLED`; with FS disabled, any `File` attachment fails before any filesystem access.
- Open with no-follow semantics where the platform supports it; then `fstat` the opened descriptor; require a regular file.
- Reject: symlink, directory, FIFO, device, socket, unreadable file.
- Read at most the per-file cap + 1 byte to detect growth and overflow.
- Never echo the absolute path in public errors — use the caller-supplied filename or a redacted marker.
- Buffer attachments (`Buffer`/`Uint8Array`) are copied before async work.
- No URL, stream, glob, or directory attachment form exists — an SSRF/local-file-read vector by design.
- Aggregate raw attachment bytes are checked before MIME composition; wire size is checked after composition.

## 3. Filesystem test fixtures

| Fixture | Expected |
| --- | --- |
| regular file within cap / exactly at cap | accepted |
| regular file at cap + 1, or growing during read | `LIMIT_ATTACHMENT_BYTES` |
| symlink to regular file / directory / FIFO / device | rejected |
| missing path | validation/configuration error |
| unreadable file | configuration error, path redacted |
| file access without opt-in | `CONFIG_FILE_ACCESS_DISABLED` |

## 4. Hard caps (DoS bounds — enforce before DATA)

- 100 recipients → `LIMIT_RECIPIENTS`
- 10 MiB per attachment → `LIMIT_ATTACHMENT_BYTES`
- 20 MiB aggregate raw attachments → `LIMIT_ATTACHMENT_TOTAL`
- 30 MiB serialized wire size → `LIMIT_WIRE_BYTES`
- Timeouts connect 10 s / operation 30 s / total 120 s / close 125 s → `TIMEOUT_*`
- One active send per transport (immediate `BUSY`); one connection per send, dropped after the transaction; bounded close that cannot hang; no unbounded buffering or collection growth.

Oversized inputs must fail **before connect** — validation order guarantees it (see sendust-core-semantics §2).
