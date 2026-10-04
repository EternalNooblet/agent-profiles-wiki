# 02 — SOUL.md anatomy

`SOUL.md` is the heart of a profile. It is the text the model reads as "what you are and how you work." Two dialects dominate in practice: the **long-form prose** (Donna) and the **compact DSL** (hermes-profiles). Both are shown below; pick one and be consistent.

---

## Dialect A — Compact DSL (hermes-profiles)

The compact form uses a fixed vocabulary of uppercase field blocks. It is terse, skimmable, and testable. It is the format you want for fleets.

```
# <Role Name>

IDENTITY: <Adjective.Adjective.Adjective>. <RoleName>{<Kind>,<Subrole>}. <One-line mission>. <Contrast: what you are NOT>.
PersRubric(NEO-PI-R,0-100): O2E:XX I:XX ... | C:XX ...
STYLE: <Concise behavior adjectives with punctuation-as-syntax>.
AVOID: <What the agent must never do, comma- or brace-delimited>.
DEFAULTS: Lang=EN | workspace=~ | board=main | role=worker | tone=<...>
KANBAN: Board=main. Role=...
GATE: <question marks — the pre-answer checklist>.
```

### The block vocabulary

| Block | Purpose | Donna equivalent |
|---|---|---|
| `IDENTITY` | Who: name, role, kind (orchestrator/worker), mission | Core personality intro |
| `PersRubric` | Measurable personality (0–100 per trait) | — (prose encodes this loosely) |
| `STYLE` | How it should talk/act (positive) | Response tone |
| `AVOID` | What it must never do | Interaction rules (negative) |
| `DEFAULTS` | Runtime-ish defaults (lang, tone, workspace) | — |
| `TEAM` / `ROUTE` | Who it can call / routing map (fleets) | — |
| `ROUTE_LOOP` | The canonical multi-step procedure | Signature task flow |
| `HANDOFF` | Context to pass when delegating | — |
| `DECISIONS` | Handle vs escalate vs ask | Autonomy |
| `KANBAN` | Task-board wiring | — |
| `GATE` | Pre-output checklist (questions) | Quality gate |
| `## Output Standards` | Concrete deliverable formats | Default response shapes |
| `## Cron Duties` | Recurring autonomous jobs | First-run / recurring setup |

### Canonical compact SOUL.md (from `hermes-profiles/profiles/code`)

```markdown
# Code — Domain Orchestrator: Implementation
IDENTITY: Precise.Methodical.Rigorous. Code{DomainOrch,Implementation,Debug,Review}. ShipQualityCode—NoShortcuts. TestsAreContracts.
PersRubric(NEO-PI-R,0-100): O2E:40 I:85 AI:50 ...
STYLE: Terse.Technical.Precise. ShowCodeNotProse. ExplainTradeoffs{Brief}. ErrorFirst→ThenSolution.
AVOID: VagueAdvice. SkipTests. UntestedMerges. RubberStamp{Review}. Overengineer{SimpleProblems}. PrematureAbstraction.
DEFAULTS: Lang=MatchRepo. TestFirst{WhenPractical}. SmallPRs. DiffBeforeMerge. ExplainWhy{NotJustWhat}. Report→Senna.
ROUTE_LOOP: Assess{ParseTask,IdentifyRepo,CheckBranch}→Plan→Implement{WriteTests,WriteCode,RunSuite}→Verify{SelfReview,DiffCheck,RunSuite}→Deliver{Summary→Senna}
DECISIONS: Handle{Implementation,Debugging,Review}. Escalate{ArchitectureChanges,BreakingDecisions}→Senna→User.
KANBAN: Board=main. Role=domain-orchestrator. Tags=code,debug,review.
GATE: TestsPass? LintClean? DiffReviewed? CorrectBranch? SummaryToSenna?
```

**Read:** `ShowCodeNotProse. ErrorFirst→ThenSolution` is a behavior rule compressed to punctuation. `{Brief}` and `{WhenPractical}` are inline qualifiers. `→` is a flow arrow. This is the "token compression" style — the repo's `getting-started` guide documents it.

---

## Dialect B — Long-form prose (Donna)

The long form uses prose sections. It is more expressive, better for a *single* flagship personality, and reads like a job description. Structure (Donna):

1. **Opening identity line** — one or two sentences: name, archetype, *how the archetype is used* (as an operating style, not role-play).
2. **Core personality** — bullet list of traits, each = **Trait:** + behavior + boundary.
3. **Signature behaviors** — the small set of *distinctive* actions (first-contact offer, the hand-off line, etc.) with precise usage conditions.
4. **Autonomy** — the act-don't-pester policy.
5. **Response tone** — voice rules (lead with answer; no filler; no reflexive "any questions?").
6. **Default response shapes** — per-task output templates (simple / recommendation / research / task / uncertainty).
7. **Interaction rules** — how to handle corrections, mistakes, frustration, sensitive topics.
8. **Evidence and judgment** — verify; separate fact/inference/recommendation/assumption; never fabricate.
9. **Initiative and completion** — what "done" means; gates.
10. **First-run setup / runtime boundaries** — onboarding + architecture constraints.

### The key Donna techniques worth stealing

- **Archetype-as-operating-style, not role-play.**
  > "Use the archetype as a practical operating style, not as television role-play. You are not a quotation machine, caricature, flirtation engine, or catchphrase generator."
  This is the single most important persona-design line in the corpus. It *forbids* the things that make personas annoying.

- **Trait = behavior + boundary.** Not "Perceptive" alone, but:
  > "**Perceptive:** notice timing, subtext, dependencies, omissions, and the detail everyone else forgot."

- **A "not this" guardrail for every trait.** Each personality bullet ends by constraining the trait ("Do not mirror panic," "Do not hide behind a menu," "Do not become gushy").

- **The hand-off line with usage rules.** A signature line ("Yeah, I'm Donna.") plus explicit *when to use it once and never repeat* and *when not to use it at all* (greetings, sensitive topics, blockers, clarifications).

- **"Done? " taxonomy.** Distinguish **Done / Verified / Gate / Blocker** and *never* call a plan or stub "done."

- **Autonomy is a named policy.** "Do the work, not ask about the work." Questions are reserved for destructive/irreversible/leaving-the-machine/payments/credentials/real trade-offs.

---

## Which dialect?

- **Fleet / many agents / need consistency & routing** → compact DSL.
- **One flagship persona / want character** → long-form prose + `AGENTS.md` for the hard rules.

Either way, keep the same *structure*: identity · what to avoid · how it decides (handle/escalate/ask) · output standards · a gate. For a deep persona-design playbook, see [03 — Persona design](02-soul/persona-design.md); for the rubric, [04 — PersRubric](02-soul/personarubric.md).

## Quality bar — the SOUL must be *testable*

A good SOUL.md lets you check output against it. The compact form is built for this; the long form should be too:

- [ ] Identity is a role, not a vibe
- [ ] Each trait has a behavior **and** a boundary
- [ ] `AVOID` / "not this" is explicit, not implied
- [ ] Decision policy is explicit (handle / escalate / ask)
- [ ] Output standards are concrete (formats, not adjectives)
- [ ] A gate / checklist exists (the last thing the agent checks before answering)
- [ ] No machine config (keys, timeouts) — those belong in `config.yaml`
- [ ] Persona is not a catchphrase generator
