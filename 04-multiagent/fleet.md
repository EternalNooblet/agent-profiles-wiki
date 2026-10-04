# 08 — Orchestrator vs worker (fleets)

The hermes-profiles repo's deepest insight: **effective systems are fleets, not monoliths.** One agent doing everything is a single point of failure for quality. Split *concerns* into agents and route.

## The core pattern: orchestrator + workers

From the fleet layout in `hermes-profiles`:

```
team:
  senna        # orchestrator — takes tasks, decides, delegates, reports
  research     # worker — deep research, returns findings to Senna
  business     # worker — market analysis, returns to Senna
  code         # orchestrator — implementation (sub-fleet)
  media        # worker — content / visual generation
  ...
```

- **Orchestrator** = the *user-facing* agent. It takes the task, breaks it down, routes to the right worker(s), composes the final answer, and reports back.
- **Worker** = a *specialist*. It does one kind of work well and reports *to the orchestrator*, not to the user.

```
User ──▶ [Orchestrator: Senna]
                 │  routes by domain
        ┌────────┼─────────┬──────────┬─────────┐
        ▼        ▼         ▼          ▼
     [Research] [Business][Code]   [Media]
        │        │         │         │
        └────────┴─────────┴─────────┘
                 │
                 ▼
              Senna composes → User
```

## Why fleets beat monoliths

| Concern | Monolith | Fleet |
|---|---|---|
| Consistency | One persona does everything (muddy) | Each agent is a distinct, testable kind |
| Tool surface | Everything (noisy, drift-prone) | Narrow per role (least privilege) |
| Tuning | Change one SOUL for everything | Change one SOUL for one role |
| Routing | Implicit, ad-hoc | Explicit (`ROUTE` block) |
| Scale | Bloats as you add capabilities | Composes linearly |

## The orchestrator's job (Senna pattern)

An orchestrator SOUL defines:

- **`TEAM:`** who exists and what each does (a one-line capability per worker).
- **`ROUTE:`** the routing map — *which domain goes to which worker*.
- **`HANDOFF:`** what context to pass when delegating (the orchestrator must hand off *enough* that the worker can act).
- **`DECISIONS:`** handle vs. escalate vs. ask.
- **`GATE:`** what to verify before sending a result back to the user.

The orchestrator is the only agent that talks to the user. Workers *report to the orchestrator*; the orchestrator *synthesizes* and reports to the user.

## The worker's job

A worker SOUL defines:

- A narrow **`IDENTITY`** (the role, the mission).
- `STYLE` / `AVOID` for *that* role.
- A `ROUTE_LOOP` / signature task flow (e.g., the research loop).
- `Output Standards` — a concrete format for what it returns *to the orchestrator*.
- A `GATE` — a pre-return checklist.

Workers are **not** user-facing. They don't chat; they produce deliverables.

## The three routing layers

1. **Static (config-level):** the tool surface available to the profile.
2. **Dynamic (routing):** `ROUTE` block — which worker handles which domain.
3. **Adaptive (in-task):** the orchestrator's `DECISIONS` / `GATE` — when to actually route vs. handle itself.

This is "lean architecture" in its full form: broad surface, *deterministic first-turn routing*, adaptive within the task.

## When to build a fleet vs. one agent

| Situation | Recommendation |
|---|---|
| A single task type, one user | One agent (long-form persona, e.g. Donna) |
| Multiple *domains* the user spans (research + code + business) | Fleet — orchestrator + per-domain workers |
| A moderation / narrow-powers bot | One agent, minimal plugins (the `gamehub-mod` pattern) |
| You want *testability* across agents | Fleet, all compact DSL, shared rubric |

## The fleet rule

> **An orchestrator is only as good as its routing.** If `ROUTE` is vague, the fleet degrades into "ask the orchestrator what to do."

Every fleet profile should answer, in one line: "Given task X, which worker do I route to, and what do I hand off?" If the SOUL can't, the routing is broken.

See [09 — Routing & handoff](04-multiagent/routing.md) for the routing patterns in detail.
