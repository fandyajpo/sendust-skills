---
name: sendust-evidence
description: Report Sendust work with correct evidence labels, the canonical command matrix, gate rules, and the full task report template. Use when completing a task, running or requesting validation, or judging phase-gate approval.
---

# Sendust evidence and reporting

Normative source: `docs/engineering/ai-implementation-contract.md` §6–§9, `docs/engineering/toolchain-baseline.md`, `docs/planning/definition-of-done.md`, and `docs/planning/quality-gates.md` in the sendust repository.

**See also:** [sendust-engineering](../sendust-engineering/SKILL.md) for the workflow whose report this skill formats · [sendust-docs-sync](../sendust-docs-sync/SKILL.md) for status-model rules · [sendust-knowledge-base](../sendust-knowledge-base/SKILL.md) when validating RFC compliance claims.

## 1. Evidence levels — use exactly these labels

- **IMPLEMENTED** — source/config/documentation changes are present in the inspected repository state.
- **STATICALLY REVIEWED** — manually inspected; no execution implied.
- **EXECUTED PASS** — a command or test actually ran and returned success; include exact command, environment, commit, and log/evidence.
- **EXECUTED FAIL** — a command ran and failed; include failure evidence.
- **NOT RUN** — execution was unavailable or intentionally not performed.
- **BLOCKED** — a prerequisite, unresolved decision, or failure prevents safe progress.

## 2. Forbidden claims

Never write "tests pass", "build succeeds", "compiles", "lint clean", "runtime supported", or "gate approved" based only on source inspection. A failed or unexecuted required check stays pending; never convert it to PASS or N/A for convenience. "Runtime supported" additionally requires blocking CI on the exact runtime × OS × CPU × ABI/libc combination — a version pin is not a support claim.

## 3. Gate rules

- Writing code is not validating code. A task with executable acceptance criteria cannot be COMPLETED without the required execution evidence.
- The agent that authored an implementation must not approve its own phase gate unless the gate's reviewer and evidence rules are satisfied.
- An implementation-only agent may mark deliverables IMPLEMENTED/STATICALLY REVIEWED but must leave execution-dependent acceptance criteria pending and must never mark a gate APPROVED.
- Every phase must record exact commands, working directory, required tool versions, and expected result — never guessed commands or flags.
- Flaky tests are quarantined with a risk ID and cannot count as gate evidence; do not mask flakiness with retries.

## 4. Canonical command matrix (repository root; exact tool pins from toolchain-baseline)

| Concern | Command | Expected |
| --- | --- | --- |
| Rust formatting | `cargo fmt --all -- --check` | exit 0, no diff |
| Rust lint | `cargo clippy --workspace --all-targets -- -D warnings` | exit 0, no warning |
| Core-only unit tests (no Node) | `cargo test -p sendust-core --all-targets` | exit 0 |
| Native binding compile | `cargo check -p sendust-napi --all-targets` | exit 0 |
| Workspace Rust tests | `cargo test --workspace --all-targets` | exit 0 |
| Architecture check | `cargo metadata --format-version 1 --no-deps` then `node scripts/check-architecture.mjs` | two Rust packages; boundaries valid |
| Docs structure check | `node scripts/check-docs.mjs` | links, IDs, counts valid |
| JS install | `npm ci` | reproducible install |
| TS lint / format / typecheck | `npm --workspace ./ts run lint` / `format:check` / `typecheck` | exit 0 |
| TS tests | `npm --workspace ./ts run test` | Vitest exit 0 |
| Node runtime smoke (P05 deliverable) | `node tests/runtime/smoke.mjs` | addon loads; fake-relay send and close pass |
| Bun runtime smoke (P05) | `bun run tests/runtime/smoke.mjs` | same contract |
| Deno runtime smoke (P05) | `deno run --allow-ffi --allow-read tests/runtime/smoke.mjs` | same contract, local node_modules |

Dependency evidence commands: `cargo tree -e features`, `cargo tree -d`, `npm ls --all`. All commands are NOT RUN until actually executed; passing compilation alone is not license, advisory, or feature-policy evidence.

## 5. Required CI scope (the sendust repository must provide)

Rust formatting and lint · Rust core tests without Node linkage · N-API crate compile and binding tests · TypeScript typecheck, lint, tests · architecture/dependency-boundary checks · fake SMTP/TLS tests with loopback-only networking · the exact Node.js/Bun/Deno runtime matrix · platform-specific native artifact build/load tests · security and release evidence.

## 6. Required task report format

```text
Task: Pxx-00y — <title>
Status: IMPLEMENTED / STATICALLY REVIEWED / NOT RUN / BLOCKED

Prerequisites:
- <task id>: <evidence link and status>

Files:
- <path>: <change and reason>

Implementation:
- <behavior/interface delivered>

Traceability:
- Requirements: <IDs>
- Constraints: <IDs>
- ADRs: <IDs>
- Modules: <owners>

Static review:
- <checks performed>
- <findings or none>

Execution:
- Tests: NOT RUN | EXECUTED PASS | EXECUTED FAIL
- Build: NOT RUN | EXECUTED PASS | EXECUTED FAIL
- Lint/format: NOT RUN | EXECUTED PASS | EXECUTED FAIL
- Runtime/platform: NOT RUN | EXECUTED PASS | EXECUTED FAIL

External validation:
- `<exact command>` — run from `<directory>`; expected evidence: <artifact>

Remaining risks/blockers:
- <items>

Next eligible task:
- <task id and reason>
```
