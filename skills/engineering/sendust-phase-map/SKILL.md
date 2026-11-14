---
name: sendust-phase-map
description: Locate work in the Sendust plan — all 19 phases with status and gate tasks, parallelism rules, task ID conventions, and which document owns what. Use when picking the next task, checking prerequisites, or reporting phase status.
---

# Sendust phase map

> Scope: this skill only serves building Sendust — the standalone native email delivery engine (Rust core → native Node-API addon → thin TS facade). Anything else (SMTP server, hosted service, queue, alternate transport, pure-JS engine) is out of scope.

Sources in the sendust repository: `docs/planning/phase-index.md`, `master-roadmap.md`, `task-registry.md`, `dependency-graph.md`, `milestones.md`, and the phase plans in `docs/phases/`.

**See also:** [sendust-task-workflow](../sendust-task-workflow/SKILL.md) for how to execute a task · [sendust-evidence-and-gates](../../quality/sendust-evidence-and-gates/SKILL.md) for gate and status rules · [sendust-docs-sync](../../documentation/sendust-docs-sync/SKILL.md) for the status model.

## 1. Which document owns what

| Document | Owns |
| --- | --- |
| `docs/product/requirements.md` | product behavior |
| Accepted ADRs (`docs/architecture/decisions/`) | architecture decisions |
| `docs/phases/phase-XX.md` | task-level implementation detail |
| `docs/planning/task-registry.md` | index of all tasks and prerequisites |
| `docs/planning/dependency-graph.md` | phase DAG and critical path |
| `docs/planning/quality-gates.md` | gate checklists |
| `docs/planning/definition-of-done.md` | evidence rules |

The registry and dependency graph index work; they must not silently redefine it.

## 2. Phase table

P00–P03 are COMPLETED with approved gate evidence; A01 (multi-runtime) and A02 (modular monolith) amendments are approved at design level; P04–P18 not started.

| Phase | Name | Gate task | Status |
| --- | --- | --- | --- |
| P00 | Discovery | P00-005 | COMPLETED |
| P01 | Research | P01-005 | COMPLETED |
| P02 | Architecture | P02-005 | COMPLETED |
| P03 | Repository baseline | P03-005 | COMPLETED |
| P04 | Rust core (errors, validation, message model, state machine, SMTP orchestration) | P04-005 | NOT STARTED |
| P05 | Binding (Node-API 8, owned memory) | P05-005 | NOT STARTED |
| P06 | SMTP (relay sessions, TLS/auth, recipient policy) | P06-005 | NOT STARTED |
| P07 | MIME (headers, alternative bodies, deterministic serialization) | P07-005 | NOT STARTED |
| P08 | Attachments (opt-in files, caps) | P08-005 | NOT STARTED |
| P09 | Public API (TypeScript transport, send, close, errors) | P09-005 | NOT STARTED |
| P10 | Reliability (races, timeouts, cleanup, fault injection) | P10-005 | NOT STARTED |
| P11 | Security (threat-model audit, redaction) | P11-005 | NOT STARTED |
| P12 | QA (traceability, contract tests) | P12-005 | NOT STARTED |
| P13 | Performance (budgets, benchmarks) | P13-005 | NOT STARTED |
| P14 | Distribution (platform matrix, clean installs) | P14-005 | NOT STARTED |
| P15 | Documentation (quick start, lifecycle) | P15-005 | NOT STARTED |
| P16 | Release engineering (versioning, reproducible build) | P16-005 | NOT STARTED |
| P17 | RC and MVP approval | P17-005 | NOT STARTED |
| P18 | Post-MVP | P18-005 | NOT STARTED |

## 3. Task conventions

- Every phase has exactly five tasks: `Pxx-001` … `Pxx-005`.
- Task IDs are stable and must not be renumbered after work begins.
- The next eligible task after the current state was P04-001 (core error taxonomy and deterministic validation order).
- Every phase plan has the same 11 sections: objective, scope, tasks, technical implementation details, dependencies, risks and edge cases, testing requirements, acceptance criteria, deliverables, completion gate, and status.

## 4. Parallelism rules

The phase DAG is mostly sequential. Genuine parallel work:

- **P07 (MIME)** can run with **P06 (SMTP)** after P04 — both consume core types, not each other.
- **P15 (documentation)** can begin after P09 and run beside P10–P14, but P15-004 requires P11-005 and P15-005 requires all documentation tasks.
- **P14-001–P14-003** can start after P05-005/P03-005 beside P09–P13; only P14-005 is gated on P12-005 and P13-005.
- Within P01, P01-004 (Node/Node-API research) could run beside P01-001–P01-003 after P00-005.

A task may start only when its registry prerequisites are complete with evidence.
