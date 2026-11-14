---
name: sendust-knowledge-base
description: Find and use authoritative technical references for Sendust — Rust, Tokio, TLS, SMTP/MIME RFCs, email authentication, DNS, Node-API, Deno, Bun, TypeScript, packaging, testing, and supply chain — with a phase-to-reference map and the 15 normative implementation guidelines. Use when researching protocol behavior, crate APIs, or runtime compatibility before making architectural decisions.
---

# Sendust technical knowledge base

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Sendust learns through this reference catalog: `references.md` in this skill folder lists every approved knowledge source, organized into 17 domains. Use it whenever a task requires looking up how something actually works.

**See also:** [sendust-task-workflow](../../engineering/sendust-task-workflow/SKILL.md) for how a task selects its reading · [sendust-crate-and-module-boundaries](../../architecture/sendust-crate-and-module-boundaries/SKILL.md) for the architecture these references constrain · [sendust-threat-register](../../security/sendust-threat-register/SKILL.md) for the security envelope · [sendust-evidence-and-gates](../../quality/sendust-evidence-and-gates/SKILL.md) for RFC-compliance evidence rules.

## 1. Source-of-truth hierarchy

When sources disagree, resolve in this order (consistent with the sendust AI implementation contract):

1. **RFC Editor and IANA** — normative for email protocol standards (SMTP, MIME, SPF, DKIM, DMARC, ARC, DNS) and registered values (media types, root zones).
2. **Official Rust documentation** — the Book, Reference, std docs, rustc/cargo/rustdoc, Clippy, rustfmt — for language and toolchain behavior.
3. **Official runtime documentation** — Node.js, Deno, Bun, TypeScript — for compatibility and API behavior of each runtime.
4. **Crate documentation and repositories** (lettre, mail-send, mail-builder, mail-parser, mail-auth, napi, rustls, tracing, criterion, …) — implementation references only. They never replace RFC specifications.

## 2. Phase → reference map

| Phase | Domains to consult | Key references |
| --- | --- | --- |
| P04 Rust core | 1, 2, 3 | Rust Book/Reference/std, tokio, rustls/ring, RFC 5321/5322, RFC 3463 |
| P05 Binding | 8, 9, 10, 11 | Node N-API/addons, napi-rs, Deno FFI + node compat, Bun FFI/node APIs |
| P06 SMTP | 4, 6 | RFC 5321, 6409, 4954, 3207, 7504, 6531, 2920, 3030, 3463, 1870; native client protocol; lettre/mail-send as references only; provider docs for interop |
| P07 MIME | 5 | RFC 2045–2049, 2183, 2231, 2387, 2392, 3676; IANA media types; mail-builder/mail-parser as references |
| P08 Attachments | 5, 3 | RFC 2183/2231; secrecy/zeroize for bounded handling; filesystem fixtures |
| P09 Public API | 11, 8 | TypeScript handbook (modules, declarations, publishing), Node packages/ESM |
| P10 Reliability | 2, 16 | tokio time/futures, proptest, mockall, fault-injection policy |
| P11 Security | 3, 6, 17 | rustls/webpki/ring, RFC 7672/8460/8461, OWASP, RustSec, threat model AB-01..AB-14 |
| P12 QA | 16 | testing strategy, cargo-llvm-cov, mail-parser (test-only) |
| P13 Performance | 2, 16 | criterion, tokio, performance strategy budgets |
| P14 Distribution | 12, 8 | npm publishing, provenance, Node addons packaging, Bun/Deno publish |
| P15 Documentation | 11, 9, 10 | runtime-specific docs for consumer examples |
| P16 Release | 12, 17 | SLSA, SBOM/provenance, cargo-deny/audit |
| P17–P18 | — | release checklist, feedback intake |

Match the domain to the task at hand; do not consult everything for every task.

## 3. How to use a reference

- Evaluate relevance to the current phase and task; do not blindly implement every feature a document mentions.
- Verify the current status of every RFC before relying on it: obsolete, updated, or superseded.
- Verify crate versions, feature flags, and runtime compatibility against the pinned versions in `docs/engineering/toolchain-baseline.md` and the admission rules in `docs/engineering/dependency-policy.md` in the sendust repository.
- Do not assume every SMTP extension is universally supported by every provider; provider docs (SES, Gmail, Exchange, Resend, Postmark, Mailgun, SendGrid, Brevo) are behavioral references for interoperability, not standards.
- Validate protocol behavior through automated tests and interoperability tests, and record evidence per the sendust-evidence-and-gates skill. Never claim RFC compliance without corresponding test evidence.
- Pinned dependencies are napi 3.16.2, rustls 0.23, and tokio 1.x; lettre and mail-send remain implementation references only — not dependencies (ADR-0017 supersedes ADR-0002's lettre choice).

## 4. Implementation guidelines (normative)

1. Rust documentation is the source of truth for Rust language and runtime behavior.
2. RFC Editor and IANA are the normative sources for email protocol standards and registered values.
3. Official runtime documentation is the source of truth for Node.js, Deno, Bun, and TypeScript compatibility.
4. Third-party crate documentation and repositories are implementation references, not replacements for RFC specifications.
5. Verify the current status of every RFC, including whether it is obsolete, updated, or superseded.
6. Do not assume every SMTP extension is universally supported by every provider.
7. Validate protocol behavior through automated tests and interoperability tests.
8. Keep the Rust core independent of JavaScript runtimes.
9. Keep Node.js, Deno, and Bun bindings separated from the core delivery implementation.
10. Do not introduce mandatory queues, workers, databases, or persistence into the standalone delivery core.
11. Do not silently retry failed deliveries or persist messages without explicit user configuration.
12. Every architectural decision must respect existing Sendust specifications and ADRs.
13. Never claim RFC compliance without corresponding test evidence.
14. Prefer secure defaults, explicit configuration, typed errors, and predictable resource lifecycle management.
15. Evaluate each reference according to its relevance to the current implementation phase; do not blindly implement every feature mentioned in the documentation.

Guidelines 8–11 and 14 restate constraints owned by the sendust-architecture and sendust-security skills; treat those skills and the sendust ADRs as the authority when they and this list appear to diverge.
