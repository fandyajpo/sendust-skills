# Sendust Master Technical Documentation & Knowledge Base

Purpose: build a standalone, modular, production-grade email delivery engine using Rust, with Node.js, Deno, Bun, and TypeScript ecosystem support.

All references are technical knowledge sources. Prefer official documentation and normative specifications. Verify RFC status, versions, runtime compatibility, and implementation behavior before making architectural decisions.

## 1. Rust language & systems engineering

- [The Rust Book](https://doc.rust-lang.org/book/)
- [Rust Reference](https://doc.rust-lang.org/reference/)
- [Rust std](https://doc.rust-lang.org/std/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [rustc](https://doc.rust-lang.org/rustc/)
- [Cargo](https://doc.rust-lang.org/cargo/)
- [rustdoc](https://doc.rust-lang.org/rustdoc/)
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
- [Edition Guide](https://doc.rust-lang.org/edition-guide/)
- [Clippy](https://doc.rust-lang.org/clippy/)
- [Unstable Book](https://doc.rust-lang.org/unstable-book/)
- [rustfmt](https://rust-lang.github.io/rustfmt/)
- [Rustlings](https://rust-lang.github.io/rustlings/)

## 2. Rust async, concurrency & networking

- [Tokio](https://tokio.rs/) · [Tokio tutorial](https://tokio.rs/tokio/tutorial) · [tokio docs.rs](https://docs.rs/tokio/latest/tokio/)
- [futures](https://docs.rs/futures/latest/futures/)
- [async-trait](https://docs.rs/async-trait/latest/async_trait/)
- [bytes](https://docs.rs/bytes/latest/bytes/)
- [socket2](https://docs.rs/socket2/latest/socket2/)
- [mio](https://docs.rs/mio/latest/mio/)
- [hyper](https://docs.rs/hyper/latest/hyper/)
- [reqwest](https://docs.rs/reqwest/latest/reqwest/)
- [hickory-resolver](https://docs.rs/hickory-resolver/latest/hickory_resolver/)

## 3. Rust TLS, cryptography & security

- [rustls](https://docs.rs/rustls/latest/rustls/)
- [tokio-rustls](https://docs.rs/tokio-rustls/latest/tokio_rustls/)
- [webpki](https://docs.rs/webpki/latest/webpki/)
- [ring](https://docs.rs/ring/latest/ring/)
- [aws-lc-rs](https://docs.rs/aws-lc-rs/latest/aws_lc_rs/)
- [ed25519-dalek](https://docs.rs/ed25519-dalek/latest/ed25519_dalek/)
- [rsa](https://docs.rs/rsa/latest/rsa/)
- [sha2](https://docs.rs/sha2/latest/sha2/)
- [base64](https://docs.rs/base64/latest/base64/)
- [subtle](https://docs.rs/subtle/latest/subtle/)
- [zeroize](https://docs.rs/zeroize/latest/zeroize/)
- [secrecy](https://docs.rs/secrecy/latest/secrecy/)
- [constant_time_eq](https://docs.rs/constant_time_eq/latest/constant_time_eq/)

## 4. SMTP & email protocol standards

- [RFC 5321](https://www.rfc-editor.org/info/rfc5321/) — SMTP
- [RFC 5322](https://www.rfc-editor.org/info/rfc5322/) — Internet Message Format
- [RFC 6409](https://www.rfc-editor.org/info/rfc6409/) — Message Submission (587)
- [RFC 4954](https://www.rfc-editor.org/info/rfc4954/) — SMTP AUTH
- [RFC 3207](https://www.rfc-editor.org/info/rfc3207/) — STARTTLS
- [RFC 7504](https://www.rfc-editor.org/info/rfc7504/) — SMTP Security via TLS
- [RFC 6531](https://www.rfc-editor.org/info/rfc6531/) — SMTPUTF8
- [RFC 6532](https://www.rfc-editor.org/info/rfc6532/) — Internationalized Email Headers
- [RFC 3461](https://www.rfc-editor.org/info/rfc3461/) — DSN extension
- [RFC 3463](https://www.rfc-editor.org/info/rfc3463/) — Enhanced status codes
- [RFC 3464](https://www.rfc-editor.org/info/rfc3464/) — Delivery Status Notifications
- [RFC 3030](https://www.rfc-editor.org/info/rfc3030/) — CHUNKING/BDAT
- [RFC 2920](https://www.rfc-editor.org/info/rfc2920/) — PIPELINING

## 5. MIME & email message construction

- [RFC 2045](https://www.rfc-editor.org/info/rfc2045/) · [RFC 2046](https://www.rfc-editor.org/info/rfc2046/) · [RFC 2047](https://www.rfc-editor.org/info/rfc2047/) · [RFC 2049](https://www.rfc-editor.org/info/rfc2049/) — MIME
- [RFC 2183](https://www.rfc-editor.org/info/rfc2183/) — Content-Disposition
- [RFC 2231](https://www.rfc-editor.org/info/rfc2231/) — parameter value encoding
- [RFC 2387](https://www.rfc-editor.org/info/rfc2387/) — multipart/related
- [RFC 2392](https://www.rfc-editor.org/info/rfc2392/) — cid: and mid: URLs
- [RFC 3676](https://www.rfc-editor.org/info/rfc3676/) — text/plain format=flowed
- [IANA media types registry](https://www.iana.org/assignments/media-types/media-types.xhtml)

## 6. Email authentication & deliverability

### SPF

- [RFC 7208](https://www.rfc-editor.org/info/rfc7208/)

### DKIM

- [RFC 6376](https://www.rfc-editor.org/info/rfc6376/) — DKIM
- [RFC 8301](https://www.rfc-editor.org/info/rfc8301/) — sha256, relaxed restrictions
- [RFC 8463](https://www.rfc-editor.org/info/rfc8463/) — Ed25519-SHA256

### DMARC

- [RFC 9989](https://www.rfc-editor.org/info/rfc9989/) · [RFC 9990](https://www.rfc-editor.org/info/rfc9990/) · [RFC 9991](https://www.rfc-editor.org/info/rfc9991/)

### ARC

- [RFC 8617](https://www.rfc-editor.org/info/rfc8617/)

### SMTP TLS security

- [RFC 8460](https://www.rfc-editor.org/info/rfc8460/) — TLSRPT
- [RFC 8461](https://www.rfc-editor.org/info/rfc8461/) — MTA-STS
- [RFC 7672](https://www.rfc-editor.org/info/rfc7672/) — DANE for SMTP

### Email abuse reporting

- [RFC 5965](https://www.rfc-editor.org/info/rfc5965/) — ARF
- [RFC 6591](https://www.rfc-editor.org/info/rfc6591/) — Authentication-Failure reporting
- [RFC 6650](https://www.rfc-editor.org/info/rfc6650/) — feedback loop report handling

## 7. DNS & domain infrastructure

- [RFC 1034](https://www.rfc-editor.org/info/rfc1034/) · [RFC 1035](https://www.rfc-editor.org/info/rfc1035/) — DNS
- [RFC 3596](https://www.rfc-editor.org/info/rfc3596/) — AAAA
- [RFC 2782](https://www.rfc-editor.org/info/rfc2782/) — SRV
- [RFC 5890](https://www.rfc-editor.org/info/rfc5890/) · [RFC 5891](https://www.rfc-editor.org/info/rfc5891/) — IDNA
- [RFC 9460](https://www.rfc-editor.org/info/rfc9460/) — SVCB/HTTPS
- [RFC 6698](https://www.rfc-editor.org/info/rfc6698/) — TLSA/DANE
- [IANA root database](https://www.iana.org/domains/root/db)

## 8. Node.js & native addons

- [Node.js API](https://nodejs.org/docs/latest/api/) · [Node.js learn](https://nodejs.org/en/learn)
- [packages](https://nodejs.org/api/packages.html) · [modules](https://nodejs.org/api/modules.html) · [ESM](https://nodejs.org/api/esm.html) · [TypeScript](https://nodejs.org/api/typescript.html)
- [N-API](https://nodejs.org/api/n-api.html) · [Addons](https://nodejs.org/api/addons.html)
- [Test runner](https://nodejs.org/api/test.html) · [Diagnostics](https://nodejs.org/en/learn/diagnostics) · [Security best practices](https://nodejs.org/en/learn/getting-started/security-best-practices)
- [napi-rs](https://napi.rs/) · [napi crate](https://docs.rs/napi/latest/napi/)

## 9. Deno

- [Deno docs](https://docs.deno.com/) · [Runtime](https://docs.deno.com/runtime/) · [CLI reference](https://docs.deno.com/runtime/reference/cli/) · [Deno API](https://docs.deno.com/api/deno/)
- [TypeScript](https://docs.deno.com/runtime/fundamentals/typescript/) · [Node compatibility](https://docs.deno.com/runtime/fundamentals/node/) · [Security](https://docs.deno.com/runtime/fundamentals/security/)
- [Testing](https://docs.deno.com/runtime/fundamentals/testing/) · [Benchmarking](https://docs.deno.com/runtime/fundamentals/benchmarking/) · [Workspaces](https://docs.deno.com/runtime/fundamentals/workspaces/)
- [Publishing](https://docs.deno.com/runtime/packages/publishing/) · [JSR](https://jsr.io/docs)

## 10. Bun

- [Bun docs](https://bun.com/docs) · [Runtime](https://bun.com/docs/runtime) · [API](https://bun.com/docs/api)
- [TypeScript](https://bun.com/docs/runtime/typescript) · [Node.js APIs](https://bun.com/docs/runtime/nodejs-apis) · [FFI](https://bun.com/docs/runtime/ffi)
- [Package manager](https://bun.com/docs/pm) · [Workspaces](https://bun.com/docs/pm/workspaces) · [Lockfile](https://bun.com/docs/pm/lockfile) · [publish](https://bun.com/docs/pm/cli/publish)
- [bun test](https://bun.com/docs/test) · [Bundler](https://bun.com/docs/bundler)

## 11. TypeScript

- [Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) · [All docs](https://www.typescriptlang.org/docs/)
- [tsconfig reference](https://www.typescriptlang.org/tsconfig/)
- [Modules](https://www.typescriptlang.org/docs/handbook/modules/introduction.html)
- [Declaration files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html) · [Publishing](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html)
- [Project references](https://www.typescriptlang.org/docs/handbook/project-references.html) · [Type compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html) · [Utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [Compiler API](https://github.com/microsoft/TypeScript/wiki/Using-the-Compiler-API)

## 12. Package managers & distribution

### pnpm

- [Docs](https://pnpm.io/) · [CLI](https://pnpm.io/cli) · [Workspaces](https://pnpm.io/workspaces) · [Catalogs](https://pnpm.io/catalogs)
- [.npmrc](https://pnpm.io/npmrc) · [Lockfile](https://pnpm.io/lockfile) · [patch](https://pnpm.io/cli/patch) · [publish](https://pnpm.io/cli/publish)
- [Filtering](https://pnpm.io/filtering) · [pnpmfile](https://pnpm.io/pnpmfile) · [Supply chain security](https://pnpm.io/supply-chain-security)

### npm

- [Docs](https://docs.npmjs.com/) · [CLI](https://docs.npmjs.com/cli/) · [package.json](https://docs.npmjs.com/cli/configuring-npm/package-json) · [Workspaces](https://docs.npmjs.com/cli/using-npm/workspaces)
- [Publishing unscoped](https://docs.npmjs.com/creating-and-publishing-unscoped-public-packages/) · [Publishing scoped](https://docs.npmjs.com/creating-and-publishing-scoped-public-packages/)
- [npm config](https://docs.npmjs.com/cli/v11/using-npm/config) · [Scripts](https://docs.npmjs.com/cli/using-npm/scripts)
- [Security auditing](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities) · [Provenance statements](https://docs.npmjs.com/generating-provenance-statements)

## 13. Rust email implementation references

These are implementation references, not normative protocol specifications.

### Lettre

- [lettre on GitHub](https://github.com/lettre/lettre) · [lettre docs.rs](https://docs.rs/lettre/latest/lettre/)

### Stalwart Mail Send

- [mail-send on GitHub](https://github.com/stalwartlabs/mail-send) · [mail-send docs.rs](https://docs.rs/mail-send/latest/mail_send/)

### Stalwart Mail Components

- [mail-builder](https://docs.rs/mail-builder/latest/mail_builder/) · [mail-parser](https://docs.rs/mail-parser/latest/mail_parser/) · [mail-auth](https://docs.rs/mail-auth/latest/mail_auth/) · [mail-auth on GitHub](https://github.com/stalwartlabs/mail-auth)

### Nodemailer

- [Nodemailer on GitHub](https://github.com/nodemailer/nodemailer)

## 14. Email provider documentation

- [AWS SES](https://docs.aws.amazon.com/ses/) · [Gmail SMTP/IMAP](https://developers.google.com/workspace/gmail/imap/imap-smtp) · [Exchange Server](https://learn.microsoft.com/en-us/exchange/)
- [Resend](https://resend.com/docs) · [Postmark](https://postmarkapp.com/developer) · [Mailgun](https://documentation.mailgun.com/) · [SendGrid](https://www.twilio.com/docs/sendgrid) · [Brevo](https://developers.brevo.com/)

## 15. Observability & telemetry

- [tracing](https://docs.rs/tracing/latest/tracing/) · [tracing-subscriber](https://docs.rs/tracing-subscriber/latest/tracing_subscriber/)
- [OpenTelemetry](https://opentelemetry.io/docs/) · [OpenTelemetry Rust](https://opentelemetry.io/docs/languages/rust/) · [Prometheus](https://prometheus.io/docs/introduction/overview/)

## 16. Testing, fuzzing & benchmarking

- [Rust testing (Book ch. 11)](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [proptest](https://docs.rs/proptest/latest/proptest/) · [cargo-fuzz book](https://rust-fuzz.github.io/book/)
- [criterion](https://docs.rs/criterion/latest/criterion/) · [miri](https://github.com/rust-lang/miri) · [cargo-llvm-cov](https://github.com/taiki-e/cargo-llvm-cov)
- [mockall](https://docs.rs/mockall/latest/mockall/) · [wiremock](https://docs.rs/wiremock/latest/wiremock/)

## 17. Security & supply chain

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/) · [OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/)
- [NIST CSRC](https://csrc.nist.gov/)
- [RustSec](https://rustsec.org/) · [OSV](https://osv.dev/) · [cargo-audit](https://docs.rs/cargo-audit/latest/cargo_audit/) · [cargo-deny](https://docs.rs/cargo-deny/latest/cargo_deny/)
- [SLSA](https://slsa.dev/) · [GitHub code security](https://docs.github.com/en/code-security)
