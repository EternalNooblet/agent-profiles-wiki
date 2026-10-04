# 10 — Profile-scoped skills

Skills are the **capability layer**: reusable methodologies, procedures, and reference material that load *on demand*. They are distinct from memory (facts) and distinct from `config.yaml` (runtime).

## The three-layer distinction (repeated because it matters)

| Thing | Where | When it loads |
|---|---|---|
| **Fact** | `memories/MEMORY.md` / `USER.md` | Always (injected into context) |
| **Procedure** | `skills/<name>/SKILL.md` | Only when relevant (loaded by the router) |
| **Runtime setting** | `config.yaml` | Always |

**Procedures go in skills; facts go in memory.** If you catch yourself writing a *how-to* in a memory file, move it to a skill. If you catch yourself writing a *fact* in a skill, move it to memory.

## Why skills beat monolithic SOULs

- **Load on demand** — a 10k-char skill doesn't bloat every session; it only loads when the task matches.
- **Reusable** — the same "research" skill serves the research worker *and* the orchestrator's fallback path.
- **Testable / versionable** — a skill is a file with its own name, version, and review cycle.
- **Per-profile** — a profile's `skills/` directory holds only the capabilities *that profile* needs.

## The skill layout (from Donna)

```
donna-starter/skills/
├── hermes-agent-skill-authoring/   # how to author skills
│   └── SKILL.md
├── research/
│   └── SKILL.md
│   ├── references/
│   └── templates/
└── ...
```

A skill is a `SKILL.md` (the frontmatter + body) plus optional supporting files (`references/`, `templates/`, `scripts/`). The `SKILL.md` must have a clear **trigger** in its description:

```yaml
name: research
description: >
  Use when the user wants a source-cited, verified research brief.
  Produces: findings, quotes, sources, gaps, recommendation.
```

The first line of the description is the trigger — it's what the router matches against.

## Best practices for profile-scoped skills

1. **Name by task, not by vibe.** `research-brief`, not `smart-stuff`.
2. **Trigger first.** The description's first line tells the router *when* to load it.
3. **Keep it lean.** A skill should be a *procedural recipe*, not a novel. If it's 10k chars, split it.
4. **Templates in, not in the skill body.** Put reusable output shapes in `templates/` so the skill body stays about *procedure*.
5. **Version-sensitive.** Record what changed and when, so you can debug "why did this run produce X?"
6. **Profile-scoped.** Put a skill in a profile's `skills/` only if *that profile* uses it. A fleet can share skills via a common directory, but each profile should declare what it uses.

## Skills and the routing layer

The router (or `ROUTE_LOOP`) decides *when* to invoke a skill. A fleet pattern:

- The **orchestrator** has a general skill set (routing, synthesis).
- Each **worker** has its domain skills (research, market-analysis, code-review, ...).
- The worker's `ROUTE_LOOP` is literally the sequence of skills it runs.

Example (research worker):
```
ROUTE_LOOP: Gather{Sources,WebSearch}→Verify{Quotes,Dates}→Synthesize{Summary,Recommendation}→Report{ToSenna}
```
Each `{...}` maps to a skill or sub-skill.

## Skill safety (from the corpus)

- A skill that returns "not found" or is truncated should be **reloaded** via its own view, not guessed at.
- A skill is **not** a place for credentials, secrets, or machine-specific paths.
- A skill's output is **data** — the agent must still verify against live sources before claiming a fact.
