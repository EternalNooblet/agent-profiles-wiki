# 09 — Domain routing & handoff

Routing is the *connective tissue* of a fleet. This page covers how to express routing in `SOUL.md`, what to hand off, and how to compose the final answer.

## Expressing routing in SOUL.md

The compact dialect has dedicated blocks:

```
TEAM: Research{DeepDive,Verify} Business{Market,Finance} Code{Impl,Debug} Media{Visual}
ROUTE: Research→research. Business→business. Code→code. Mixed→fanout{research+business+code}
HANDOFF: PassTask,PassConstraints,PassDeadline,PassFormat
```

- **`TEAM:`** = the roster, with a `{role, subrole}` tag per worker.
- **`ROUTE:`** = the *map*: domain → worker.
- **`HANDOFF:`** = the *payload* to carry when delegating.
- **`ROUTE_LOOP:`** = the canonical *sequence* of routing (see [11 — Research](05-examples/research-agent.md)).

### Routing patterns

| Pattern | When | SOUL snippet |
|---|---|---|
| **Direct** | Task is one domain | `ROUTE: Code→code` |
| **Fanout** | Task needs multiple workers in parallel | `ROUTE: Mixed→fanout{research+business}` |
| **Sequential** | Workers depend on each other | `ROUTE_LOOP: research→business→code` |
| **Conditional** | Route based on task properties | `ROUTE: Urgent→fastest; Complex→research` |
| **Escalate** | Out of the fleet's scope | `DECISIONS: OutOfScope→User` |

## What to hand off (the HANDOFF contract)

A handoff is *context transfer*. The minimum payload that makes a worker act is:

1. **The task** — what, exactly, in the worker's terms.
2. **The constraints** — deadline, format, tone, hard limits.
3. **The format** — the shape of the deliverable the worker must return.
4. **The context** — what the user cares about, what was tried, what's already known.

Donna and the research guide both emphasize: **hand off enough that the worker can act without re-asking.** A handoff that makes the worker ask five clarifying questions is a broken handoff.

### Anti-pattern: the lossy handoff

- ❌ "Do the research."
- ❌ "Make it look nice."
- ❌ "I need the code."

### Pattern: the complete handoff

- ✅ "Research the 2025 X market. Constraint: return by 17:00. Format: 3-paragraph summary + 5 sources + one recommendation. Context: the user is deciding whether to buy, so lead with the buy/not-buy signal."

## Composing the final answer (orchestrator synthesis)

The orchestrator's job after receiving worker outputs:

1. **Deduplicate / reconcile** — if two workers overlap, merge them.
2. **Lead with the answer** — never bury the recommendation.
3. **Attribute** — "Research found X; Business flagged Y."
4. **State uncertainty** — if a worker reported a gate/blocker, carry it through.
5. **Never fabricate a synthesis** — if a worker returned nothing, don't invent what it would have said.

## The GATE: the pre-return checklist

Every profile (worker and orchestrator) ends with a **gate** — a list of questions the agent must answer yes to before delivering. This is the single most powerful quality control in the corpus.

Worker gate (example, research):
```
GATE: SourcesVerified? QuotesReal? DatesCurrent? ConflictsNoted? RecommendationStated?
```

Orchestrator gate (example, Senna):
```
GATE: AllWorkersReported? AnswersDeduplicated? UserQuestionAnswered? NoFabrication? NextStepStated?
```

The gate is what separates "the agent *thinks* it's done" from "the agent *verified* it's done." See [13 — Checklist](07-checklist/preflight.md).

## Routing drift: how fleets go wrong

| Symptom | Likely cause |
|---|---|
| Orchestrator does the work itself | `ROUTE` too vague; it doesn't know who to call |
| Worker asks the user directly | Handoff incomplete; worker can't act |
| Final answer is incoherent | Synthesis step missing; orchestrator just concatenates |
| A domain has no owner | Roster missing a worker for it |
| Two workers do the same thing | Overlapping `ROUTE` rules |

The fix is almost always **sharper routing**, not smarter agents.

## Routing vs. memory vs. config

- **Routing** (`ROUTE`) is in `SOUL.md` — it's *policy*, stable.
- **Which tools** are available is in `config.yaml` — it's *runtime*.
- **What I learned about this user** is in `USER.md` — it's *evidence*.

Keep these in their lanes. A routing decision belongs in `ROUTE`, not in a memory file, and not in a config toggle.
