# Templates index

## SOUL dialects
- **Compact DSL** ([02 — SOUL anatomy](02-soul/soul-structure.md)) — for fleets. Fixed uppercase field blocks (`IDENTITY`, `PersRubric`, `STYLE`, `AVOID`, `DEFAULTS`, `TEAM`, `ROUTE`, `HANDOFF`, `DECISIONS`, `KANBAN`, `GATE`, `Output Standards`, `Cron Duties`). Terse, skimmable, testable.
- **Long-form prose** ([03 — Persona design](02-soul/persona-design.md)) — for one flagship. Prose sections: identity · core personality · signature behaviors · autonomy · tone · response shapes · interaction rules · evidence · done-taxonomy · first-run.

## Rubric
- **PersRubric** ([04 — Personality rubric](02-soul/personarubric.md)) — NEO-PI-R-style 0–100 vector. The fleet's consistency + tunability mechanism.

## Dialect choice matrix

| If... | Use |
|---|---|
| One agent, daily use, want a *person* | Long-form prose + `AGENTS.md` + skin (Donna) |
| Many agents, need consistency/routing | Compact DSL + shared rubric + routing (hermes-profiles) |
| One domain specialist | Compact DSL + `Output Standards` + `Cron Duties` |
| Narrow-powers bot (moderation) | Compact DSL + scope/enforcement block + minimal plugins |

## Copy-paste starting points
- **Full starter set:** [templates](06-templates/templates.md) — `SOUL-compact`, `SOUL-detailed`, `AGENTS.md`, `config.yaml`, `MEMORY.md`, `USER.md`, `skin.yaml`.

## Worked examples (read before you build)
- **Research agent** ([11](05-examples/research-agent.md)) — the full specialized-worker build.
- **Donna Starter** ([12](05-examples/donna-starter.md)) — the annotated flagship-persona build.

## Checklists
- [13 — Preflight & maintenance checklist](07-checklist/preflight.md) — run before you ship and after every edit.
