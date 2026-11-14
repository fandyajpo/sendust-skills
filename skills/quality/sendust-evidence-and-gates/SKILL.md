---
name: sendust-evidence-and-gates
description: Report Sendust work honestly — the six evidence labels, forbidden claims, gate approval rules, and the full task report template. Use when finishing a task, claiming completion, or evaluating a phase gate.
---

# Sendust evidence and gates

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Normative source: `docs/engineering/ai-implementation-contract.md` §6–§9 and `docs/planning/definition-of-done.md` in the sendust repository.

**See also:** [sendust-validation-commands](../sendust-validation-commands/SKILL.md) for the exact commands evidence must cite · [sendust-task-workflow](../../engineering/sendust-task-workflow/SKILL.md) for the workflow whose report this formats · [sendust-docs-sync](../../documentation/sendust-docs-sync/SKILL.md) for the status model behind status updates.

## 1. Evidence levels — use exactly these labels

- **IMPLEMENTED** — source/config/documentation changes are present in the inspected repository state.
- **STATICALLY REVIEWED** — manually inspected; no execution implied.
- **EXECUTED PASS** — a command or test actually ran and returned success; include exact command, environment, commit, and log/evidence.
- **EXECUTED FAIL** — a command ran and failed; include failure evidence.
- **NOT RUN** — execution was unavailable or intentionally not performed.
- **BLOCKED** — a prerequisite, unresolved decision, or failure prevents safe progress.

## 2. Forbidden claims

Never write "tests pass", "build succeeds", "compiles", "lint clean", "runtime supported", or "gate approved" based only on source inspection. A failed or unexecuted required check stays pending; never convert it to PASS or N/A for convenience. "Runtime supported" requires blocking CI on the exact runtime × OS × CPU × ABI/libc combination — a version pin is not a support claim.

## 3. Gate rules

- Writing code is not validating code. A task with executable acceptance criteria cannot be COMPLETED without the required execution evidence.
- The agent that authored an implementation must not approve its own phase gate unless the gate's reviewer and evidence rules are satisfied.
- An implementation-only agent may mark deliverables IMPLEMENTED/STATICALLY REVIEWED but must leave execution-dependent acceptance criteria pending and must never mark a gate APPROVED.
- Every phase must record exact commands, working directory, required tool versions, and expected result — never guessed commands or flags.
- Flaky tests are quarantined with a risk ID and cannot count as gate evidence; do not mask flakiness with retries.
- A task is not a phase gate. Never approve a gate merely because code was written.

## 4. Required task report format

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
