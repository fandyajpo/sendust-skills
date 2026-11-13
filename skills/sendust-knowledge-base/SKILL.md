---
name: sendust-knowledge-base
description: Find and use authoritative technical references for Sendust — Rust, Tokio, TLS, SMTP/MIME RFCs, email authentication, DNS, Node-API, Deno, Bun, TypeScript, packaging, testing, and supply chain. Use when researching protocol behavior, crate APIs, or runtime compatibility before making architectural decisions.
---

# Sendust technical knowledge base

Sendust learns through this reference catalog: `references.md` in this skill folder lists every approved knowledge source, organized into 17 domains. Use it whenever a task requires looking up how something actually works.

## Source-of-truth hierarchy

When sources disagree, resolve in this order (consistent with the sendust AI implementation contract):

1. **RFC Editor and IANA** — normative for email protocol standards (SMTP, MIME, SPF, DKIM, DMARC, ARC, DNS) and registered values (media types, root zones).
2. **Official Rust documentation** — the Book, Reference, std docs, rustc/cargo/rustdoc, Clippy, rustfmt — for language and toolchain behavior.
3. **Official runtime documentation** — Node.js, Deno, Bun, TypeScript — for compatibility and API behavior of each runtime.
4. **Crate documentation and repositories** (lettre, mail-send, mail-builder, mail-parser, mail-auth, napi, rustls, tracing, criterion, …) — implementation references only. They never replace RFC specifications.

## How to use a reference

- Match the reference to the current phase. Evaluate relevance to the task at hand; do not blindly implement every feature a document mentions.
- Verify the current status of every RFC before relying on it: obsolete, updated, or superseded.
- Verify crate versions, feature flags, and runtime compatibility against the current pinned versions in `docs/engineering/toolchain-baseline.md` and `docs/engineering/dependency-policy.md` in the sendust repository.
- Do not assume every SMTP extension is universally supported by every provider; provider docs (SES, Gmail, Exchange, Resend, Postmark, Mailgun, SendGrid, Brevo) are behavioral references for interop, not standards.
- Validate protocol behavior through automated tests and interoperability tests, and record the evidence per the sendust-evidence skill. Never claim RFC compliance without corresponding test evidence.

## Implementation guidelines (normative)

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

Guidelines 8–11 and 14 restate architecture constraints owned by the sendust-architecture and sendust-security skills; treat those skills and the sendust ADRs as the authority when they and this list appear to diverge.
