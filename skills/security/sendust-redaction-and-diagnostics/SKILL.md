---
name: sendust-redaction-and-diagnostics
description: Prevent Sendust diagnostics from leaking sensitive data — the redaction table for errors and logs, the panic policy, and FFI memory safety. Use when touching logging, errors, panic handling, or the FFI boundary.
---

# Sendust redaction and diagnostics

Sources in the sendust repository: `docs/engineering/security-strategy.md` §7–§8, `docs/architecture/error-model.md` §6–§7, ADR-0008 (owned memory), ADR-0013 (error taxonomy). Related abuse cases: AB-07, AB-11, AB-12 (see [sendust-threat-register](../sendust-threat-register/SKILL.md)).

**See also:** [sendust-tls-and-transport-security](../sendust-tls-and-transport-security/SKILL.md) for credential handling in AUTH · [sendust-input-validation-and-caps](../sendust-input-validation-and-caps/SKILL.md) for path redaction in attachment errors.

## 1. Redaction rules (verified by marker scans)

| Data | Error message | `details` | Logs |
| --- | --- | --- | --- |
| Password / auth token / AUTH challenge material | never | never | never |
| Full message body | never | never | never |
| Attachment bytes | never | never | never |
| Absolute file path | never | never | never — redact to basename or `<redacted>` |
| Username | redacted/omitted | omitted | omitted |
| Host and port | allowed | allowed | allowed |
| SMTP numeric response code and server text | allowed if credential-free | allowed | allowed |
| Caller-supplied attachment filename | allowed | allowed | allowed |
| Timeout category and elapsed budget | allowed | allowed | allowed |
| Recipient address supplied by the caller | allowed | allowed | allowed (not in bulk) |

- `details` may include `smtpCode`, `smtpEnhancedCode`, `host`, `port`, `timeoutMs`, `elapsedMs`, coarse `stage`, and `recipient` for a validation error tied to one address. It must never include passwords, full bodies, attachment bytes, tokens, or absolute paths.
- Error message strings are never a contract; callers branch on `result.status` and `error.code`.
- Redaction is verified by serializing errors and scanning for marker values (P09-003, P11-004). A redaction regression is a security gate failure.

## 2. Panic policy

- Recoverable failures return `Result`; panics are reserved for violated internal invariants.
- `catch_unwind` is applied at exported napi boundaries where the panic strategy allows it; `panic = "abort"` is not used for the addon.
- A caught panic becomes `INTERNAL` with a generic message; the panic payload is never exposed.
- Panics on detached Tokio tasks must not leave a transport permanently BUSY — the state machine releases the slot on task completion or abort (tested in P10-001).

## 3. FFI and memory safety

- No borrowed JS memory crosses an `await`. Every `Buffer`, `ArrayBuffer`, typed array, and string is synchronously copied into Rust-owned memory before async work is scheduled (AB-07).
- Do not retain `BufferSlice`, raw N-API pointers, or JS-backed views across an `await`.
- Strings are UTF-8 validated on conversion; invalid UTF-16 is rejected, not lossily replaced.
- Numbers convert to bounded integers with range checks; no silent truncation.
- `unsafe` blocks are minimized, justified, and audited.
- The binding never creates a Tokio runtime; napi-rs manages it.
- Blocking file reads run on a blocking pool (`tokio::task::spawn_blocking` or equivalent), never on the reactor thread.
- The transport object is retained for the lifetime of in-flight work via explicit ownership, not GC reachability.
