# Agent Kickoff Prompt

Copy everything below the line into your AI agent's first message (or into the codebase's `AGENTS.md` / `CLAUDE.md` so every session starts with it).

---

You are working on **Sendust** (https://github.com/fandyajpo/sendust) — a standalone native email delivery engine: a runtime-independent Rust core, a native Node-API addon (`crates/sendust-napi`), and a thin TypeScript facade, embeddable by Node.js, Bun, and Deno applications.

Your work is governed by the agent skills at https://github.com/fandyajpo/sendust-skills. Follow them in this order:

1. Read the skills repo's `README.md` first — it states the mission (build the standalone native engine) and the in/out-of-scope lists. Nothing outside that scope is ever the goal.
2. Read `skills/engineering/sendust-task-workflow/SKILL.md` (required reading order, authority precedence, workflow) and `skills/engineering/sendust-phase-map/SKILL.md` (which task to pick, prerequisites, phase status).
3. For each task, additionally load the skill that matches it:
   - Changing module structure or adding a dependency → `skills/architecture/sendust-crate-and-module-boundaries` and `skills/architecture/sendust-dependency-policy`
   - Implementing `send()`/`close()`/validation/MIME/errors → `skills/architecture/sendust-core-semantics`
   - Implementing the SMTP transport → `skills/architecture/sendust-smtp-client-protocol`
   - Touching TLS/AUTH → `skills/security/sendust-tls-and-transport-security`
   - Touching validation/caps/attachments → `skills/security/sendust-input-validation-and-caps`
   - Touching logs/errors/panic/FFI → `skills/security/sendust-redaction-and-diagnostics`
   - Anything security-sensitive → `skills/security/sendust-threat-register`
   - Updating any documentation → `skills/documentation/sendust-docs-sync`
   - Claiming completion → `skills/quality/sendust-evidence-and-gates`
   - Needing a protocol/runtime reference → `skills/references/sendust-knowledge-base`

**Architectural direction (binding):** the SMTP transport is a hand-written native client over tokio 1 + rustls 0.23 + webpki-roots — **no lettre and no third-party SMTP library anywhere**. This is ADR-0017 superseding ADR-0002. If the sendust repository still documents lettre as the transport (ADR-0002), your first task is to record ADR-0017 there (superseding ADR-0002) with synchronized dependency policy, toolchain baseline pins, P06 phase plan, and gate updates — per the repo's own `docs/engineering/ai-implementation-contract.md`. Remove any lettre dependency from `Cargo.toml`/`Cargo.lock` as part of the revamp. Never silently keep lettre "just for now".

**Operating rules:**

- Never claim "tests pass", "compiles", or "runtime supported" without actually executing it. Use the exact evidence labels (IMPLEMENTED / STATICALLY REVIEWED / EXECUTED PASS / EXECUTED FAIL / NOT RUN / BLOCKED) and the task report template from `sendust-evidence-and-gates`.
- One task at a time, chosen from the phase map's current state; check prerequisites with evidence before starting.
- No placeholders, no `todo!`, no fake success paths, no scope drift toward queues/servers/alternate transports.
- Revamping existing code counts as a task: inspect the current skeleton, diff it against `sendust-core-semantics` and `sendust-crate-and-module-boundaries`, list every deviation (wrong validation order, lettre imports, `any` in TypeScript, missing state-machine transitions, etc.), then fix them task by task with the same evidence discipline.

Start by reading the skills repo and the sendust repository's current state, then produce: (1) the current-state audit, (2) the list of deviations from the skills, (3) the ordered task plan, (4) the first task executed per the workflow.
