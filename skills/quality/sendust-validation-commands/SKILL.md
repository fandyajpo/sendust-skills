---
name: sendust-validation-commands
description: Run the exact validation Sendust requires — the canonical command matrix with expected results, dependency evidence commands, and mandatory CI scope. Use when preparing validation instructions, CI evidence, or external validation steps.
---

# Sendust validation commands

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Source: `docs/engineering/toolchain-baseline.md` §4–§5 in the sendust repository — the single source of truth for tool pins and canonical commands. All commands run from the repository root; JavaScript scripts run in the `./ts` npm workspace. Exact pins: Rust 1.100.0 (MSRV 1.88.0, edition 2024), Node.js 22.23.5 / 24.22.3 / 26.13.1, Bun 1.4.4, Deno 2.9.8, npm 12.2.2, TypeScript 7.0.2, Biome 2.6.7, Vitest 5.1.0.

**See also:** [sendust-evidence-and-gates](../sendust-evidence-and-gates/SKILL.md) for how to label the results · [sendust-dependency-policy](../../architecture/sendust-dependency-policy/SKILL.md) for what the dependency commands prove.

## 1. Canonical command matrix

| Concern | Command | Expected |
| --- | --- | --- |
| Rust formatting | `cargo fmt --all -- --check` | exit 0, no diff |
| Rust lint | `cargo clippy --workspace --all-targets -- -D warnings` | exit 0, no warning |
| Core-only unit tests (no Node) | `cargo test -p sendust-core --all-targets` | exit 0 |
| Native binding compile | `cargo check -p sendust-napi --all-targets` | exit 0 |
| Workspace Rust tests | `cargo test --workspace --all-targets` | exit 0 |
| Architecture check | `cargo metadata --format-version 1 --no-deps` then `node scripts/check-architecture.mjs` | exactly two Rust packages; dependency direction and forbidden dependencies valid |
| Docs structure check | `node scripts/check-docs.mjs` | links, IDs, counts, pin claims valid |
| JS install | `npm ci` | reproducible install from the committed root lockfile |
| TypeScript lint | `npm --workspace ./ts run lint` | Biome exit 0 |
| TypeScript format check | `npm --workspace ./ts run format:check` | Biome exit 0, no diff |
| TypeScript typecheck | `npm --workspace ./ts run typecheck` | TypeScript exit 0, strict |
| TypeScript unit/contract tests | `npm --workspace ./ts run test` | Vitest exit 0 |
| Node runtime smoke (P05 deliverable) | `node tests/runtime/smoke.mjs` | native addon loads; fake-relay send and close pass |
| Bun runtime smoke (P05 deliverable) | `bun run tests/runtime/smoke.mjs` | same observable contract |
| Deno runtime smoke (P05 deliverable) | `deno run --allow-ffi --allow-read tests/runtime/smoke.mjs` | same contract, using local `node_modules` |

The three runtime smoke entry points are P05 deliverables; they do not exist before P05 and are deliberately absent from P03 CI.

## 2. Dependency evidence commands

```text
cargo metadata --format-version 1 --no-deps
cargo tree -e features
cargo tree -d
node scripts/check-architecture.mjs
node scripts/check-docs.mjs
npm ci
npm ls --all
```

Passing compilation alone is not license, advisory, or feature-policy evidence.

## 3. CI and reproducibility rules

- CI uses the exact Rust, Node.js, Bun, Deno, npm, Biome, TypeScript, and Vitest versions above; no floating `latest`, `lts`, or `stable` tags.
- Native runtime jobs use a matrix generated from the toolchain baseline.
- One committed `package-lock.json` is authoritative; `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb` must not be committed.
- Rust dependencies use exact versions for architecture-critical crates plus a committed `Cargo.lock`; CI resolves with `--locked`.
- A toolchain version update requires a reviewed PR, compatibility results, and synchronized baseline + dependent doc updates.
- Never silently substitute a command: if a canonical command cannot be implemented as written, the owning task must propose a precise replacement and update the baseline, phase plan, and CI contract in the same change.

## 4. Mandatory CI scope (the sendust repository must provide)

Rust formatting and lint · Rust core tests without Node linkage · N-API crate compile and binding tests · TypeScript typecheck, lint, tests · architecture/dependency-boundary checks · fake SMTP/TLS tests with loopback-only networking · the exact Node.js/Bun/Deno runtime matrix · platform-specific native artifact build/load tests · security and release evidence.

Platform values (minimum glibc, macOS deployment version, Windows MSVC toolset, runner images, cross-compile digests) are frozen only by P14-003 from actual build images — they are never guessed or claimed before clean-install tests.
