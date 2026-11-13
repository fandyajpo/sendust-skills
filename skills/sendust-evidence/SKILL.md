---
name: sendust-evidence
description: Report Sendust work with correct evidence labels, validation records, and gate rules. Use when completing a task, running or requesting validation, or judging phase-gate approval.
---

# Sendust evidence and reporting

Normative source: `docs/engineering/ai-implementation-contract.md` §6–§9, `docs/planning/definition-of-done.md`, and `docs/planning/quality-gates.md` in the sendust repository.

## Evidence levels — use exactly these labels

- **IMPLEMENTED** — source/config/documentation changes are present in the inspected repository state.
- **STATICALLY REVIEWED** — manually inspected; no execution implied.
- **EXECUTED PASS** — a command or test actually ran and returned success; include exact command, environment, commit, and log/evidence.
- **EXECUTED FAIL** — a command ran and failed; include failure evidence.
- **NOT RUN** — execution was unavailable or intentionally not performed.
- **BLOCKED** — a prerequisite, unresolved decision, or failure prevents safe progress.

## Forbidden claims

Never write "tests pass", "build succeeds", "compiles", "lint clean", "runtime supported", or "gate approved" based only on source inspection. A failed or unexecuted required check stays pending; never convert it to PASS or N/A for convenience.

## Gate rules

- Writing code is not validating code. A task with executable acceptance criteria cannot be COMPLETED without the required execution evidence.
- The agent that authored an implementation must not approve its own phase gate unless the gate's reviewer and evidence rules are satisfied.
- An implementation-only agent may mark deliverables IMPLEMENTED/STATICALLY REVIEWED but must leave execution-dependent acceptance criteria pending and must never mark a gate APPROVED.
- Runtime/platform support cannot be claimed until exact runtime × OS × CPU × ABI/libc combinations pass blocking CI.

## Validation command records

Each phase must list exact commands, working directory, required tool versions, and expected result. Do not guess commands or tool flags. Required CI scope for the sendust repository: Rust format/lint, Rust core tests without Node linkage, N-API compile and binding tests, TypeScript typecheck/lint/tests, architecture and dependency-boundary checks, fake SMTP/TLS tests with loopback-only networking, the exact Node.js/Bun/Deno runtime matrix, platform-specific native artifact build/load tests, and security/release evidence. Exact versions and canonical commands live in `docs/engineering/toolchain-baseline.md`.

## Required task report format

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
