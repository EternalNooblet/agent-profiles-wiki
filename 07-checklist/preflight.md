# 13 — Preflight & maintenance checklist

Two checklists: **preflight** (before you ship a profile) and **maintenance** (after every edit). Run both — the corpus's top failure modes all show up as unchecked boxes.

## Preflight (before the first real task)

### Identity
- [ ] `SOUL.md` exists and states the role as an operating style (not role-play)
- [ ] Each trait has a behavior **and** a boundary
- [ ] An explicit "avoid" / "not this" list exists
- [ ] Decision policy is explicit: handle / escalate / ask
- [ ] Output standards are concrete (formats, not adjectives)
- [ ] A `GATE` (pre-answer checklist) exists

### Separation of layers
- [ ] Identity is in `SOUL.md` (not `config.yaml`)
- [ ] Hard policy is in `AGENTS.md` (not `SOUL.md`)
- [ ] No provider keys / secrets / machine paths in `SOUL.md` or `config.yaml`
- [ ] No task progress or imperative instructions in memory

### Security
- [ ] `allow_private_urls: false`
- [ ] Secret redaction on; redaction pattern sane
- [ ] Plugins default off; only enabled roles' plugins on
- [ ] `cron_mode: deny` unless genuinely needed
- [ ] Least-privilege tool surface (a moderation bot has no terminal)

### Memory
- [ ] `MEMORY.md` = practices; `USER.md` = preferences (split correctly)
- [ ] Memory seeded so the first session behaves consistently
- [ ] No stale TODOs or task-progress left in memory

### Routing (if in a fleet)
- [ ] `ROUTE` maps every domain to a worker
- [ ] Handoff payload is complete (task, constraints, deadline, format, context)
- [ ] Orchestrator composes + synthesizes; workers report *to the orchestrator*, not the user
- [ ] One clear owner per domain (no gaps, no overlaps)

### Appearance (optional)
- [ ] One skin per profile; readable colors
- [ ] Short, distinctive banner
- [ ] `display.skin` set in `config.yaml`

## Maintenance (after every SOUL/config edit)

| Do | Why |
|---|---|
| Move **one axis at a time** (rubric) | Isolates the cause of drift |
| **Re-test with a real task** after edits | Rubric/prose are claims; behavior is truth |
| **Declarative memory** — facts, not commands | Commands in memory get re-read as directives in later sessions |
| **Replace/consolidate** stale memory | Memory that survives compression still poisons context |
| **Verify a recalled fact** before it drives an action | Memory ≠ live truth |
| **Read the skill back** before acting on it | A truncated/pruned skill must be reloaded, not guessed |

## The three questions to answer on every failure

1. **Was it a persona problem?** (wrong trait / boundary) → edit `SOUL.md`.
2. **Was it a policy problem?** (did it do something it shouldn't?) → edit `AGENTS.md` or the gate.
3. **Was it a runtime problem?** (wrong tool / tool surface / config?) → edit `config.yaml` or the skill.

Most "the agent misbehaved" reports resolve to one of these three. The distinction keeps you from editing the wrong file.

## Run this on the profile you're about to ship

1. Open [02](02-soul/soul-structure.md) → **Quality bar** (the SOUL must be testable).
2. Open [13](07-checklist/preflight.md) → **Preflight** (all boxes).
3. Run **one real task** (not a toy) and read the output against the `GATE`.
4. Only then route production work to it.

## What "good" looks like

- The agent's *first* session already behaves consistently (seeded memory).
- The *second* session is faster (learned facts in memory).
- A *new* user with the same profile gets a coherent experience (the profile is self-contained).
- An edit to one trait doesn't break the others (the rubric is a vector, not prose).
