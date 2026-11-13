---
name: sendust-docs-sync
description: Synchronize Sendust documentation after any change — the dependent-document sync map, authority precedence, ID conventions, status model, and the documentation consistency gate. Use when updating requirements, ADRs, phase plans, the task registry, toolchain pins, API contracts, gates, or summary documents.
---

# Sendust documentation synchronization

Source of the rules: `docs/engineering/ai-implementation-contract.md` §5.5 and §10 in the sendust repository. The rule: when a source-of-truth document changes, update every affected dependent document **in the same change**. Never update a summary without updating its authoritative source; never leave stale contradictory text elsewhere.

**See also:** [sendust-engineering](../sendust-engineering/SKILL.md) for the workflow that triggers syncing · [sendust-evidence](../sendust-evidence/SKILL.md) for evidence levels that status updates must cite · [sendust-architecture](../sendust-architecture/SKILL.md) when an ADR is involved.

## 1. Authority precedence (what wins when documents conflict)

1. `toolchain-baseline.md` — the only document allowed to change a version pin.
2. `docs/product/requirements.md`, then `mvp-scope.md`, `vision.md`.
3. Accepted ADRs; a later superseding ADR beats an earlier one.
4. Architecture contracts (overview, system design, public API, data flow, error model), interpreted consistently with requirements and ADRs.
5. The task section in `docs/phases/phase-XX.md`.
6. `task-registry.md` and `dependency-graph.md`.
7. Summaries and navigation: READMEs, phase index, roadmap, matrices, strategies.

A lower-level document may add detail but cannot override a higher-level decision. A conflict between equal or higher authorities is a blocker: identify both passages, do not implement the disputed behavior, and propose a tracked correction.

## 2. Sync map

| When this changes | Update all of these dependents |
| --- | --- |
| Requirement | constraint ledger, feature matrix, architecture/ADR as needed, phase tasks, test traceability, gates, README summaries |
| ADR | ADR index, architecture contracts, constraint ledger, phase tasks, dependency graph, quality gates, risk register, relevant gate record |
| Runtime/toolchain pin | toolchain baseline, P00/P01 research provenance, P03 command matrix, CI workflows, P05/P14 runtime matrices, P15 docs, P16 release claims |
| Task or dependency | phase plan, task registry, phase index/roadmap if summarized, dependency graph, quality gates, milestone references |
| Public API | public API contract, error model, data flow, TypeScript examples, tests, feature matrix, documentation |
| Module structure | ADR-0016, system design, P03/P04/P06 task details, architecture checks, P12 audit |
| Gate | quality-gate index, gate checklist, task registry, phase plan, definition of done, gate record |

## 3. Change rule

- Requirement change → `requirements.md` and the feature matrix.
- Architecture change → a new or superseding ADR; never silently edit an accepted ADR.
- Task change → phase plan, registry, and dependency graph where applicable.
- Completion claims without evidence are invalid.

## 4. ID conventions (must stay unique and resolvable)

Constraints C-01–C-40 · functional requirements FR-001–FR-068 · non-functional NFR-001–NFR-014 · open questions OQ-01–OQ-07 · phases P00–P18 · tasks Pxx-00y (five per phase, stable, never renumbered) · decisions ADR-0001–ADR-0016 · risks R-001–R-047 · abuse cases AB-01–AB-14 · gates QG-00–QG-17 plus QG-03A/QG-A01/QG-A02 · milestones M0–M7.

## 5. Status model

| State | Meaning |
| --- | --- |
| NOT STARTED | no work begun |
| IN PROGRESS | work begun; evidence incomplete |
| BLOCKED | named blocker with owner |
| IN REVIEW | deliverable and evidence exist; gate pending |
| COMPLETED | acceptance criteria met; evidence linked |

A task is COMPLETED only with evidence per `docs/planning/definition-of-done.md`; a phase is COMPLETED only when its gate is approved. Writing code is not the same as validating code.

## 6. Automated consistency checking

The repository ships two checkers that must stay valid after every docs change (canonical commands from the toolchain baseline, run from the repository root):

- `node scripts/check-docs.mjs` — Markdown links, duplicate/missing IDs, phase/task counts, registry-to-phase title/prerequisite parity, gate-index coverage, and runtime/toolchain claims against the baseline.
- `node scripts/check-architecture.mjs` — exactly two Rust packages; dependency direction and forbidden dependencies.

These commands are contract commands; label them NOT RUN until an execution-capable environment actually runs them.

## 7. Documentation consistency gate (check before finishing any docs change)

- One canonical definition per requirement, decision, task, and gate.
- All referenced IDs exist and are unique.
- Counts and status summaries match the underlying records.
- Every internal Markdown link resolves.
- Phase dependencies agree across phase plans, task registry, phase index, and dependency graph.
- Every gate in the index has a detailed checklist and an owning task/record.
- Architecture diagrams and directory trees reflect the latest accepted ADR.
- Product scope, feature matrix, API, error model, tests, and examples do not contradict each other.
- Version and platform claims state exact status: candidate, pinned, tested, supported, or deferred.
- No implementation or validation claim without repository or evidence support.
