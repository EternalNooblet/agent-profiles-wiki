# Agent Profiles — Best Practices & Examples Wiki

A practical guide to building effective **Hermes Agent profiles**: personas, `SOUL.md` structure, runtime configs, memory, skins, multi-agent fleets, and reusable templates.

**Sources** (both ingested in full for this wiki):
- [AtlasOmnia/donna-starter](https://github.com/AtlasOmnia/donna-starter) — a single, polished "Donna" executive-assistant profile (long-form persona + Lean architecture).
- [theheavenlyd3mon/hermes-profiles](https://github.com/theheavenlyd3mon/hermes-profiles) — a fleet of 22 profiles sharing a compact SOUL format, plus guides for research and fleet composition.

> Two complementary styles are represented. Pick the one that matches your profile's complexity:
> - **Long-form prose** (Donna) — rich, narrative, high-control persona for a single flagship agent.
> - **Compact DSL** (hermes-profiles) — terse, field-driven SOUL for fleets where consistency, routing, and testability matter.

---

## Table of contents

### Conceptual
- [01 — Architecture: layers of a profile](01-architecture/overview.md)
- [02 — SOUL.md anatomy](02-soul/soul-structure.md)
- [03 — Persona design (Donna-derived)](02-soul/persona-design.md)
- [04 — The personality rubric (PersRubric)](02-soul/personarubric.md)

### Configuration
- [05 — `config.yaml` reference & best practices](03-config/config-reference.md)
- [06 — Memory: `MEMORY.md` vs `USER.md`](03-memory/memory-model.md)
- [07 — Skins & theming](03-skin/skin-theme.md)

### Multi-agent
- [08 — Orchestrator vs worker (fleets)](04-multiagent/fleet.md)
- [09 — Domain routing & handoff](04-multiagent/routing.md)
- [10 — Profile-scoped skills](04-multiagent/skills.md)

### Worked examples
- [11 — The research agent (full guide)](05-examples/research-agent.md)
- [12 — Donna Starter (annotated)](05-examples/donna-starter.md)

### Templates
- [Templates index](06-templates/README.md) — `soul-compact`, `soul-detailed`, `config.yaml`, memory, skin

### Checklists
- [13 — Preflight & maintenance checklist](07-checklist/preflight.md)

---

## The ten principles (cheat sheet)

1. **Identity lives in `SOUL.md`** — it is the source of truth. Mutable runtime settings go in `config.yaml`; long-lived facts go in memory.
2. **Keep the layers separate** — Identity (`SOUL.md`) · Behavior rules (`AGENTS.md`) · Runtime (`config.yaml`) · Memory (`MEMORY`/`USER`) · Appearance (`skin`).
3. **Make the persona concrete, not cute.** Archetype as an *operating style*, not role-play; traits named with behavior; explicit "not this" guardrails.
4. **Encode behavior measurably.** The NEO-PI-R-style `PersRubric` makes personality tunable and drift-testable, not just prose.
5. **Define what it *avoids***. The `AVOID` block is as valuable as `STYLE`.
6. **Define output standards.** Concrete deliverable formats, not vibes.
7. **Gate the risky, act on the rest.** Autonomy: pause only for destructive/irreversible/publishing/credentials/payments/real trade-offs.
8. **Least privilege by default.** Plugins disabled, `cron_mode: deny`, manual approvals, secrets redacted, `allow_private_urls: false`, no secrets in config.
9. **Verify before claiming.** Read-backs, logs, live checks. Distinguish *code-complete / test-verified / live-gated / blocked*.
10. **Lean architecture.** Broad tool surface narrowed *dynamically* (router), not statically trimmed.

---

## Quick start

1. `hermes profile create <name>` → creates `~/.hermes/profiles/<name>/` with its own config, memory, and skills.
2. Write or copy a `SOUL.md` (see [02] and [Templates]).
3. Tune `config.yaml` (see [05]).
4. Seed `memories/MEMORY.md` (practices) + `memories/USER.md` (preferences) (see [06]).
5. Optional: drop in a `skins/<name>.yaml` and set `display.skin` (see [07]).
6. **Test one real task before routing production work to it.**
7. Run the [preflight checklist](07-checklist/preflight.md).
