# Sendust Skills

Reusable agent skills for [Sendust](https://github.com/fandyajpo/sendust), a standalone native email delivery engine (Rust core, Node-API adapter, TypeScript facade, for Node.js, Bun, and Deno).

Each skill distills Sendust's normative documentation (`AGENTS.md` and `docs/` in the main repository) into an agent-consumable skill: a folder under `skills/` containing a `SKILL.md` with YAML front matter (`name`, `description`).

## Skills

| Skill | Use when |
| --- | --- |
| [sendust-engineering](skills/sendust-engineering/SKILL.md) | Implementing, planning, or reviewing any Sendust task (`Pxx-00y`) in the sendust repository |
| [sendust-architecture](skills/sendust-architecture/SKILL.md) | Changing module structure, adding dependencies, or touching crate/runtime boundaries |
| [sendust-docs-sync](skills/sendust-docs-sync/SKILL.md) | Updating any source-of-truth document (requirements, ADRs, phases, registry, gates, summaries) |
| [sendust-evidence](skills/sendust-evidence/SKILL.md) | Reporting completed work, running validation, or judging phase-gate approval |
| [sendust-security](skills/sendust-security/SKILL.md) | Implementing SMTP/TLS, attachments, logging, errors, or anything near the threat boundary |
| [sendust-knowledge-base](skills/sendust-knowledge-base/SKILL.md) | Looking up RFCs, crate APIs, runtime compatibility, or any technical reference before making decisions |

## How to use

Point an agent at the skill folder it needs (or install the whole `skills/` directory into your agent's skills location). Each `SKILL.md` is self-contained: it states the rules and names the authoritative document paths in the sendust repository to read next. Every skill also carries a **See also** line linking its companion skills, so an agent entering through one skill can navigate to the others without leaving the set.

## Authority

These skills are summaries. They never override Sendust's own documentation. If a skill and the sendust repository disagree, the sendust repository wins:

1. `AGENTS.md` and `docs/engineering/ai-implementation-contract.md` govern agent behavior.
2. `docs/product/requirements.md` owns product behavior.
3. Accepted ADRs own architecture decisions.
4. `docs/phases/phase-XX.md` owns task-level detail.

Planning baseline of the source docs: 2026-11-13. P00–P03 completed; P04–P18 not started at the time these skills were written.
