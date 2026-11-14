# Sendust Skills

Reusable agent skills for [Sendust](https://github.com/fandyajpo/sendust), a standalone native email delivery engine (Rust core, Node-API adapter, TypeScript facade, for Node.js, Bun, and Deno).

Each skill distills Sendust's normative documentation (`AGENTS.md` and `docs/` in the main repository) into an agent-consumable skill: a folder under `skills/<category>/<skill-name>/` containing a `SKILL.md` with YAML front matter (`name`, `description`). Skills are deliberately small and single-purpose, grouped by category, so an agent can jump straight to the one it needs.

## Mission — the only purpose of these skills

These skills exist to guide agents to build **Sendust as a standalone, native email delivery engine**: a runtime-independent Rust core compiled into a native Node-API addon, exposed through a thin TypeScript facade, embeddable by Node.js, Bun, and Deno applications that submit mail through a consumer-configured SMTP relay.

In scope — everything that serves building that native engine:

- Rust core behavior: validation, message model, MIME, transport state machine, SMTP orchestration.
- The native binding (`sendust-napi`), its owned-memory conversion and error mapping.
- The thin TypeScript facade, the isolated native loader, and the pinned toolchain that compiles the native artifact.
- Tests, CI evidence, security envelope, and packaging/distribution of the native artifact.

Out of scope — never the goal of this repo or the engine, in any skill or task:

- An SMTP server, MTA, hosted/SaaS email service, mailbox platform, campaign manager, or job scheduler.
- Queues, workers, persistence, retry engines, or delivery orchestration — the consuming application owns those.
- Additional or alternate transports (SES, SendGrid, HTTP APIs), plugins, or a non-native (pure JavaScript) reimplementation of the engine.

If any task or change drifts toward these, stop and re-anchor to the sendust ADRs and the [crate and module boundaries](skills/architecture/sendust-crate-and-module-boundaries/SKILL.md).

## Skills by category

### engineering — how work happens

| Skill | Use when |
| --- | --- |
| [sendust-task-workflow](skills/engineering/sendust-task-workflow/SKILL.md) | Starting, implementing, or reviewing any Sendust task (`Pxx-00y`) |
| [sendust-phase-map](skills/engineering/sendust-phase-map/SKILL.md) | Picking the next task, checking prerequisites, or reporting phase status |

### architecture — how the engine is shaped

| Skill | Use when |
| --- | --- |
| [sendust-crate-and-module-boundaries](skills/architecture/sendust-crate-and-module-boundaries/SKILL.md) | Creating or moving code between modules, or touching the TypeScript/native/core boundaries |
| [sendust-core-semantics](skills/architecture/sendust-core-semantics/SKILL.md) | Implementing or modifying `send()`, `close()`, validation, MIME, or error mapping |
| [sendust-dependency-policy](skills/architecture/sendust-dependency-policy/SKILL.md) | Adding or changing any Rust crate, npm dependency, or GitHub Action |

### documentation — keeping the sources of truth consistent

| Skill | Use when |
| --- | --- |
| [sendust-docs-sync](skills/documentation/sendust-docs-sync/SKILL.md) | Updating requirements, ADRs, phase plans, the registry, toolchain pins, API contracts, gates, or summaries |

### quality — proving the work is real

| Skill | Use when |
| --- | --- |
| [sendust-evidence-and-gates](skills/quality/sendust-evidence-and-gates/SKILL.md) | Finishing a task, claiming completion, or evaluating a phase gate |
| [sendust-validation-commands](skills/quality/sendust-validation-commands/SKILL.md) | Preparing validation instructions, CI evidence, or external validation steps |

### security — staying inside the threat envelope

| Skill | Use when |
| --- | --- |
| [sendust-tls-and-transport-security](skills/security/sendust-tls-and-transport-security/SKILL.md) | Implementing or testing TLS, STARTTLS, AUTH, or relay connections |
| [sendust-input-validation-and-caps](skills/security/sendust-input-validation-and-caps/SKILL.md) | Implementing validation, MIME inputs, attachments, or limits |
| [sendust-redaction-and-diagnostics](skills/security/sendust-redaction-and-diagnostics/SKILL.md) | Touching logging, errors, panic handling, or the FFI boundary |
| [sendust-threat-register](skills/security/sendust-threat-register/SKILL.md) | Assessing a change against the threat model or planning security phases |

### references — where to look things up

| Skill | Use when |
| --- | --- |
| [sendust-knowledge-base](skills/references/sendust-knowledge-base/SKILL.md) | Looking up RFCs, crate APIs, runtime compatibility, or any technical reference before deciding |

## How to use

Point an agent at the skill folder it needs (or install the whole `skills/` directory into your agent's skills location). Each `SKILL.md` is self-contained: it states the rules and names the authoritative document paths in the sendust repository to read next. Every skill carries a **See also** line linking its companion skills, so an agent entering through one skill can navigate to the others without leaving the set.

## Authority

These skills are summaries. They never override Sendust's own documentation. If a skill and the sendust repository disagree, the sendust repository wins:

1. `AGENTS.md` and `docs/engineering/ai-implementation-contract.md` govern agent behavior.
2. `docs/product/requirements.md` owns product behavior.
3. Accepted ADRs own architecture decisions.
4. `docs/phases/phase-XX.md` owns task-level detail.

Planning baseline of the source docs: 2026-11-13. P00–P03 completed; P04–P18 not started at the time these skills were written.
