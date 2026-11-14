---
name: sendust-threat-register
description: Check Sendust threats and their required mitigations — assets, trust boundaries, the AB-01..AB-14 abuse-case register with evidence owners, security review checkpoints, and incident response. Use when assessing a change against the threat model or planning security work.
---

# Sendust threat register

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Sources in the sendust repository: `docs/discovery/threat-model.md` (assets, boundaries, AB register), `docs/engineering/security-strategy.md` (controls, checkpoints, incident response), `docs/planning/risk-registry.md`.

**See also:** the focused skills for each mitigation — [sendust-tls-and-transport-security](../sendust-tls-and-transport-security/SKILL.md) · [sendust-input-validation-and-caps](../sendust-input-validation-and-caps/SKILL.md) · [sendust-redaction-and-diagnostics](../sendust-redaction-and-diagnostics/SKILL.md) — and [sendust-evidence-and-gates](../../quality/sendust-evidence-and-gates/SKILL.md) for how each AB row's evidence is recorded.

## 1. Assets

| Asset | Security property |
| --- | --- |
| SMTP credentials (passwords, tokens, AUTH challenge material) | confidentiality; never logged or serialized |
| Message content (subject, body, headers, recipients) | confidentiality and integrity |
| Attachments (JS buffers and local files) | confidentiality, bounded memory, integrity |
| Relay identity (hostname, port, EHLO name, certificate) | authenticity and transport integrity |
| Process resources (tasks, sockets, file descriptors, memory) | bounded use and deterministic shutdown |
| Native boundary (JS values, Rust allocations, FFI calls) | memory safety and panic containment |
| Release artifacts (.node binaries, npm packages) | integrity, provenance, target correctness |

## 2. Trust boundaries

1. Consumer application → TypeScript wrapper: caller-controlled values and configuration.
2. TypeScript wrapper → Node-API adapter: shape conversion, native handles, typed arrays and buffers.
3. Node-API adapter → Rust core: owned values only; no borrowed JS memory enters async work.
4. Rust core → local filesystem: only when explicitly enabled; the opened descriptor is revalidated.
5. Rust core → SMTP relay/network: untrusted DNS, TCP, TLS, SMTP replies, and timing.
6. CI/build → npm registry: toolchain, dependencies, generated binaries, package metadata.

Threat assumptions: the consumer supplies the relay and credentials; the relay/network are untrusted; the caller may pass adversarial input; the library is not a sandbox against a compromised host process; tests use only loopback fakes and synthetic credentials.

## 3. Abuse-case register

| ID | Abuse case | Required mitigation | Evidence owner |
| --- | --- | --- | --- |
| AB-01 | SMTP MITM or STARTTLS stripping | implicit TLS or required STARTTLS, never opportunistic fallback; hostname and chain verification | P06-002, P11-002 |
| AB-02 | expired, self-signed, wrong-host, incomplete-chain, malformed certificate | fail closed with the rustls verifier and negative fixtures | P06-005, P11-002 |
| AB-03 | CRLF/control injection in address, header, filename, or EHLO | reject control characters; reserved headers cannot be overridden | P04-002, P07-001, P11-003 |
| AB-04 | oversized recipients, attachment, or wire payload | hard caps before DATA; deterministic validation order | P04-003, P08-003 |
| AB-05 | symlink/FIFO/device/directory attachment path | filesystem opt-in; no-follow open where supported; fstat; regular files only | P08-002, P11-003 |
| AB-06 | file changes or grows during read | open-then-fstat; bounded read plus one-byte overflow probe | P08-002 |
| AB-07 | JS mutates or detaches Buffer after send call | copy Buffer/typed-array bytes synchronously before the async task is scheduled | P05-002, P11-004 |
| AB-08 | relay stalls at DNS/connect/TLS/greeting/AUTH/MAIL/RCPT/DATA/final response | outer connect budget, per-operation budget, absolute total deadline, bounded close | P10-002, P10-004 |
| AB-09 | connection drops after DATA terminator | return unknown; no automatic retry; drop connection without a DATA-state QUIT attempt | P06-004, P10-001 |
| AB-10 | concurrent send or send/close race | synchronous state reservation under a short lock; immediate BUSY; close atomically enters closing | P04-004, P10-001 |
| AB-11 | panic crosses the Node-API boundary | Result for expected failures; unwind-capable build; N-API boundary catches where supported | P04-001, P05-004, P11-004 |
| AB-12 | credentials/body/path leak through diagnostics | structured redaction; no tracing of auth or message payload; marker scans | P09-003, P11-004 |
| AB-13 | external relay or real secret used by tests | loopback-only fake relay; network-denial test; secret scan | P03-004, P12-004 |
| AB-14 | platform binary mismatch or compromised artifact | exact target matrix, checksums, SBOM, provenance, clean-install tests | P14-003, P16-002 |

## 4. Operational invariants

- No built-in queue, worker, scheduler, persistence, retry engine, or queue adapter — orchestration is caller-owned.
- One transport accepts one active send; BUSY is immediate.
- One connection per send, dropped after the transaction; nothing retained while idle.
- Every network operation has a finite timeout; close is bounded and never GC-dependent.
- No test contacts an external SMTP relay or includes real credentials.

## 5. Security review checkpoints

P01 TLS feature validation · P03 dependency/license policy, secret scanning · P05 ownership and panic safety · P06 TLS/auth behavior, fault matrix · P08 filesystem safety and caps · P09 error redaction and public surface · P11 full threat-model audit · P14 artifact and install integrity · P16 SBOM, provenance, secret scan · P17 final security sign-off before publication.

## 6. Incident response

- A security defect blocks release; it is never waived.
- If a released version has a security defect: publish a fixed version, deprecate the affected version, document the issue, and never replace binaries under the same version.
- Credential exposure in a published artifact triggers immediate deprecation and consumer notification.
- Post-incident: add a regression test and update the security strategy.

## 7. Residual risks (accepted, documented)

- webpki-roots does not trust private enterprise roots installed only in the OS — intentional MVP trade-off.
- Cross-platform no-follow file opening differs by OS; platform tests must prove the chosen implementation.
- Cancellation during DATA is conservative: unobservable final disposition yields `unknown`.
- A native package expands supply-chain impact; provenance and immutable versioned artifacts are mandatory.
