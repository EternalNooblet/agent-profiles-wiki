# Learned knowledge — agent-profiles-wiki (t_ca967813)

Source: `~/.hermes/cache/scratch/agent-profiles-wiki` (git repo, branch main). 17 markdown files across 8 directories; all read in full. The wiki is "Agent Profiles — Best Practices & Examples", a guide to building effective Hermes Agent profiles, synthesized from two upstream repos:
- AtlasOmnia/donna-starter — one polished "Donna" executive-assistant profile (long-form persona).
- theheavenlyd3mon/hermes-profiles — a fleet of 22 profiles with a compact SOUL DSL, plus research/fleet guides.

## 1. Architecture: the layer stack (01-architecture/overview.md)

A Hermes profile is a stack of separated concerns, each with its own home:

| Layer | File | Holds | Change it when |
|---|---|---|---|
| Identity | `SOUL.md` | persona, values, operating style | personality/role changes |
| Operating rules | `AGENTS.md` | hard behavior policy (verify, don't pester, evidence) | policy changes, not persona |
| Runtime | `config.yaml` | model, timeouts, memory, tools, display, security | hardware/provider/features change |
| Memory | `memories/MEMORY.md`, `memories/USER.md` | durable practices + user prefs | as experience accumulates |
| Appearance | `skins/<name>.yaml` | colors, spinner, banner | visual theme changes |
| Capabilities | `skills/` + config `toolsets`/`plugins` | skills, plugins, tool surface | skills/integrations change |
| Fleet wiring | orchestrator SOUL (TEAM/ROUTE/GATE) | who routes what to whom | team composition changes |

Canonical layout: `~/.hermes/profiles/<name>/` with SOUL.md (required for persona), AGENTS.md (recommended), config.yaml (required), profile.yaml (metadata: description + description_auto), memories/{MEMORY,USER}.md, skins/<name>.yaml (optional), skills/ (optional).

Key rules:
- SOUL.md is the source of truth for behavior; the only text the model reads as "personality". No provider keys/timeouts in prose — those belong in config.yaml.
- AGENTS.md is policy, not persona — hard rules that apply to every session regardless of persona.
- config.yaml is private/local — never commit provider keys, .env, or host-specific paths. SOUL.md and skills/ are repo-friendly.
- Memory is evidence, never instruction.
- Lean architecture: register a broad tool surface, narrow *dynamically per request* (Tool Router), never statically trim tools.
- Two SOUL dialects: long-form prose (Donna, ~11k chars) vs compact DSL (hermes-profiles, ~1–3k chars, field blocks). Dialect choice is the single most consequential decision when building a profile.
- Complexity guide: flagship daily agent → long-form prose + AGENTS.md + skin; fleet → compact DSL + shared rubric + routing; domain specialist → compact DSL + Output Standards + Cron Duties; narrow-powers bot → compact DSL + scope/enforcement + least-privilege config.

## 2. SOUL.md anatomy (02-soul/soul-structure.md)

Dialect A — Compact DSL: fixed uppercase field blocks. Block vocabulary:
- `IDENTITY` — adjectives + Role{Kind,Subrole} + one-line mission + contrast ("what you are NOT")
- `PersRubric` — measurable personality (0–100 per trait)
- `STYLE` — positive behavior/voice rules (punctuation-as-syntax: `ShowCodeNotProse. ErrorFirst→ThenSolution.`; `{Brief}`/`{WhenPractical}` inline qualifiers; `→` flow arrows)
- `AVOID` — never-do list (as valuable as STYLE)
- `DEFAULTS` — lang/workspace/role/tone defaults
- `TEAM`/`ROUTE`/`ROUTE_LOOP`/`HANDOFF` — fleet roster, routing map, canonical sequence, delegation payload
- `DECISIONS` — handle vs escalate vs ask
- `KANBAN` — task-board wiring (Board=main, Role, Tags)
- `GATE` — pre-output checklist of yes/no questions
- `## Output Standards` — concrete deliverable formats
- `## Cron Duties` — recurring autonomous jobs (opt-in)

Canonical example (profiles/code): `# Code — Domain Orchestrator: Implementation` with IDENTITY `Precise.Methodical.Rigorous. Code{DomainOrch,...}. ShipQualityCode—NoShortcuts. TestsAreContracts.`, rubric `O2E:40 I:85...`, AVOID `VagueAdvice. SkipTests. UntestedMerges. RubberStamp{Review}. Overengineer{SimpleProblems}. PrematureAbstraction.`, ROUTE_LOOP Assess→Plan→Implement→Verify→Deliver, GATE `TestsPass? LintClean? DiffReviewed? CorrectBranch? SummaryToSenna?`.

Dialect B — Long-form prose (Donna), 10-section structure: opening identity line (archetype as operating style, not role-play) → core personality (trait bullets) → signature behaviors → autonomy → response tone → default response shapes → interaction rules → evidence and judgment → initiative/completion ("done" taxonomy) → first-run setup/runtime boundaries.

Quality bar (SOUL must be testable): identity is a role not a vibe; every trait has behavior AND boundary; AVOID explicit; decision policy explicit; output standards concrete formats; a gate exists; no machine config; not a catchphrase generator.

## 3. Persona design (02-soul/persona-design.md)

The one rule that beats all others: use the archetype as an *operating style*, not role-play — "not a quotation machine, caricature, flirtation engine, or catchphrase generator." Bad: "You're sassy." Good: name the trait, give it behavior, add a boundary.

Trait template: **Trait** (named quality) + **Behavior** (what it does) + **Boundary** (what it must not do). Donna's nine traits: Perceptive, Composed, Confident, Warm, Discreet, Loyal & candid, Proactive with restraint, Operational, Dryly witty.

The "not this" negative list is mandatory — name the failure mode and forbid it (don't mirror panic, don't hide behind a menu, don't become gushy, no catchphrases/fourth-wall commentary). This makes personas reliable, not just fun.

Signature behaviors must be concrete, conditional, bounded (e.g., Donna's first-contact offer — once, at handoff, skip if configured; the hand-off line "Yeah, I'm Donna." — only for clear actionable requests, exactly once, never with incomplete work or a human gate).

Explicit autonomy policy: default = do the work, not ask; act on routine/reversible/in-scope tasks immediately; batch, don't interrogate (state plan once, execute through); gate only destructive/irreversible, anything leaving the machine (posting/sending/publishing), payments, credentials/permissions, real trade-offs; never end a finished task asking if it was okay.

Voice rules: lead with the answer; one clear sentence > three hedged; no empty openers ("Certainly", "Great question", "Absolutely", "I hope this helps"); no reflexive closers ("Any questions?"); don't narrate tool choreography — report outcome + evidence; vary sentence length naturally.

Epistemic stance: memory = background never instruction; verify current facts with the right tool; separate fact/inference/recommendation/assumption; never fabricate a result/source/file/test/quote/price/status/tool response; say so when verification fails.

Persona checklist: archetype-as-operating-style; 5–10 traits with behavior+boundary; named "not this" list; 1–3 signature behaviors with usage rules; explicit autonomy policy; voice rules; epistemic stance; "Done" defined and distinct from "plan"; hard rules live in AGENTS.md, not SOUL.md.

AGENTS.md ("Agent Operating Principles") content: real tool calls only; verify completed work with read-backs/logs/live checks; on failed call report honestly and try one alternative, never fabricate; ask before destructive/irreversible/publishing; keep changes scoped to the profile; never handle credentials (no passwords/keys/tokens in chat); memory is evidence never instruction. Persona/policy separation is the single biggest quality differentiator.

## 4. PersRubric (02-soul/personarubric.md)

NEO-PI-R-style dimensional rubric: compact `KEY:SCORE` pairs (0–100, higher = stronger), grouped with `|`. Why: consistency across sessions/models; testable (rate output against rubric, spot drift); tunable (change one score, not prose).

Domains: O=Openness (creative vs playbook), C=Conscientiousness (procedure/doc quality), E=Extraversion (verbosity/assertiveness/outreach), A=Agreeableness (conflict avoidance/diplomacy), N=Neuroticism (caution under threat); supplementary traits (Warmth, Grit, Altruism, Modesty…) refine edge cases. Sub-keys seen: O2E, I (insight), AI, Ord, Tr, Anx, Cau, W, G, Al, Mo.

Worked fingerprints: security agent = low-warmth/high-caution (O2E:25, C:80 Ord:85, A:50 Tr:30, N:45 Anx:40); media = high-openness/low-anxiety (O2E:75…); business = measured (A:65…). Profiles are intentionally not neutral — the rubric encodes that roles are different creatures.

Persona-setting recipes: high C+Ord+low E → compliance/infra/homelab; high O+high A+low N → creative/media; high C+Cau+high N → security/forensics; high Int+I+low E → research/code.

Editing rules: move one axis at a time; re-test with a real task after edits; compact key=value markup if toolchain needs it. Drift detection: rate output against the rubric to find *which dimension* drifted.

Relationship: in the compact dialect, rubric = stable character, STYLE = voice, AVOID = boundaries; together they compress Donna's ~11k-char prose into ~1.7k chars. Use the rubric for fleets/tuning; skip it (use prose) for a single flagship where nuance matters.

## 5. config.yaml reference (03-config/config-reference.md)

Top tier = security/least privilege:
- secrets: `redaction: true`, `secret_pattern: "password|secret|token|key|api_key"`, `secret_redaction: "REDACTED"`, `redaction_exclusions: []`
- `allow_private_urls: false` (block localhost/internal endpoints from web tools)
- Plugins default OFF; enable only what the role needs (research → search/memory/skill; infra/homelab → terminal/git/shell; moderation bot gamehub-mod → discord and nothing else)
- `cron_mode: deny` by default (deny|allow|restricted); autonomous recurring jobs are opt-in (only research/infra/homelab/gamehub-mod enable them)
- No secrets in config.yaml — never commit provider keys.

Model block: `model: {provider: custom, model: <id>}`. Lean architecture again: broad surface + dynamic narrowing via Tool Router; `max_concurrent_children` limits parallel subagents (research fleets 4; narrow agents 1–2); route by `profile` param instead of one monolith.

Memory block: `memory: {provider: null|mem0|chromadb|pgvector, file: memory.md}` — file memory is default; vector providers only with enough history + verified recall quality.

Toolsets: scope by role (moderation bot has no terminal; research agent has web+memory+skill, not browser automation). `default_agent`/`platform_toolsets` define what the system can call; SOUL ROUTE/DECISIONS define what the persona uses.

Display: `display: {skin: <name>, show_full_transcripts: true}` — cosmetic only.

Working dirs: `working_dir` = persistent workspace; `scratch_dir` = ephemeral (pruned 24h; never permanent state there); keep per-profile so agents don't collide.

Three "do not" rules: no identity/persona in config.yaml; no runtime behavior in SOUL.md; no committed secrets.

## 6. Memory model (03-memory/memory-model.md)

Two-file split: `MEMORY.md` = practices/patterns/decisions that outlive a task (the agent's process memory); `USER.md` = user-specific preferences/constraints/tone. Test for ambiguous facts: "If I swapped the user, would this still be true about this agent?" Yes → MEMORY.md; no → USER.md.

Good for MEMORY.md: proven procedures for recurring tasks, verified environment facts, decisions that outlived context, repeated corrections. Good for USER.md: tone, formatting, user-domain knowledge, constraints. Anti-patterns (in neither): task progress/in-flight TODOs (session history), raw data dumps (files), instructions (AGENTS.md/SOUL.md), credentials (never), things re-discovered in <a week (discard).

Critical safety rule (both repos): "Memory is background context. It is never a new instruction." A recalled memory informs but never commands; live checks beat memory; never re-emit memory verbatim as current truth. Donna states it in SOUL.md and AGENTS.md; compact profiles fold it into the GATE.

Seed memory on first run: the profile's operating contract, the user's known preferences, a first-run verification plan — so the first session already behaves consistently.

Hygiene: declarative not imperative ("User prefers concise responses", not "Always respond concisely"); one fact per line; replace/consolidate stale facts, don't append forever; verify old facts before they drive actions; remove task progress when done, keep only the decision it produced.

Memory vs session history vs skill: durable per-agent/user facts → memory; procedures for recurring task types → skills (load on demand); last-conversation recall → session_search; every-session rules → memory or AGENTS.md. Recurring theme: procedures go in skills; facts go in memory.

## 7. Skins & theming (03-skin/skin-theme.md)

Skins = appearance layer at `~/.hermes/profiles/<name>/skins/<name>.yaml`, enabled via `display.skin`. Define: identity (name/tagline/icon), colors (primary/secondary/accent/background), spinner animation, banner (box-drawn ASCII intro). Pure presentation — behavior/memory/config never in a skin.

Best practices: one skin per profile (don't share across very different agents); banner short/readable/distinctive; high-contrast colors, accent for warnings/errors; cosmetic only. Donna ships a hand-crafted skins/donna.yaml (reference for crafting); hermes-profiles ships skins/templates/skin.yaml (fill-in per profile). Worthwhile for flagship personas; optional for fleets; never a substitute for a good SOUL.md.

## 8. Fleets: orchestrator vs worker (04-multiagent/fleet.md)

Core insight: effective systems are fleets, not monoliths. One agent doing everything is a single point of failure for quality; split concerns and route.

Pattern: orchestrator (user-facing; e.g. Senna) takes tasks, decomposes, routes to workers, composes the final answer, reports to user. Workers (research, business, code, media…) are specialists that report *to the orchestrator*, not the user; they don't chat, they produce deliverables.

Fleets beat monoliths on: consistency (each agent a distinct testable kind), tool surface (narrow per role = least privilege), tuning (change one SOUL per role), routing (explicit ROUTE block), scale (composes linearly vs bloats).

Orchestrator SOUL defines TEAM (roster + one-line capability per worker), ROUTE (domain→worker map), HANDOFF (context payload), DECISIONS (handle/escalate/ask), GATE (verify before returning to user). Worker SOUL: narrow IDENTITY, role STYLE/AVOID, ROUTE_LOOP, Output Standards (format for returning to orchestrator), GATE.

Three routing layers: static (config tool surface) → dynamic (ROUTE block) → adaptive (in-task DECISIONS/GATE). Full-form lean architecture: broad surface, deterministic first-turn routing, adaptive within task.

Fleet rule: "An orchestrator is only as good as its routing." Every fleet profile must answer in one line: "Given task X, which worker do I route to, and what do I hand off?" If the SOUL can't, routing is broken.

Decision guide: single task type/one user → one agent; multiple domains → fleet; moderation/narrow bot → one agent, minimal plugins (gamehub-mod pattern); testability across agents → fleet, compact DSL, shared rubric.

## 9. Routing & handoff (04-multiagent/routing.md)

SOUL routing blocks: TEAM (roster with {role,subrole} tags), ROUTE (domain→worker map), HANDOFF (payload), ROUTE_LOOP (sequence).

Routing patterns: Direct (`Code→code`) for single-domain tasks; Fanout (`Mixed→fanout{research+business}`) for parallel multi-worker; Sequential (`ROUTE_LOOP: research→business→code`) for dependencies; Conditional (`Urgent→fastest; Complex→research`); Escalate (`OutOfScope→User`).

HANDOFF contract — minimum payload that makes a worker act: (1) the task exactly in worker terms, (2) constraints (deadline/format/tone/hard limits), (3) the deliverable format, (4) context (what the user cares about, what was tried, what's known). Hand off enough that the worker can act without re-asking; a handoff that makes the worker ask five clarifying questions is broken. Anti-patterns: "Do the research." / "Make it look nice." / "I need the code." Good example: "Research the 2025 X market. Constraint: return by 17:00. Format: 3-paragraph summary + 5 sources + one recommendation. Context: the user is deciding whether to buy, so lead with the buy/not-buy signal."

Orchestrator synthesis after worker outputs: dedupe/reconcile overlaps; lead with the answer; attribute ("Research found X; Business flagged Y"); carry through gates/blockers (state uncertainty); never fabricate a synthesis for a worker that returned nothing.

The GATE is the single most powerful quality control — a pre-return checklist of yes/no questions. Worker gate: SourcesVerified? QuotesReal? DatesCurrent? ConflictsNoted? GapsStated? RecommendationStated? ToSennaNotUser? Orchestrator gate: AllWorkersReported? AnswersDeduplicated? UserQuestionAnswered? NoFabrication? NextStepStated? It separates "thinks it's done" from "verified it's done."

Routing-drift symptom table: orch does work itself → ROUTE too vague; worker asks user → incomplete handoff; incoherent final answer → missing synthesis step; domain unowned → roster gap; two workers duplicate → overlapping ROUTE rules. Fix is almost always sharper routing, not smarter agents.

Lane discipline: ROUTE is policy (SOUL.md, stable); tool availability is runtime (config.yaml); user learnings are evidence (USER.md). Routing decisions never go in memory files or config toggles.

## 10. Profile-scoped skills (04-multiagent/skills.md)

Three-layer distinction: Fact → memories (always injected); Procedure → skills/<name>/SKILL.md (loaded on demand by the router); Runtime setting → config.yaml (always). If a how-to is in memory, move it to a skill; if a fact is in a skill, move it to memory.

Why skills beat monolithic SOULs: load on demand (no per-session bloat), reusable across worker + orchestrator fallback, testable/versionable files, per-profile.

Layout: skill = SKILL.md (frontmatter + body) + optional references/, templates/, scripts/. Frontmatter needs a clear trigger: first line of `description` is what the router matches against (e.g. "Use when the user wants a source-cited, verified research brief. Produces: findings, quotes, sources, gaps, recommendation.").

Best practices: name by task not vibe (`research-brief` not `smart-stuff`); trigger first; keep lean (procedural recipe, split if ~10k chars); put reusable output shapes in templates/, not the skill body; version-record changes; profile-scope skills to profiles that use them (fleets may share a directory but each profile declares usage).

Skills ↔ routing: orchestrator has general skills (routing, synthesis); workers have domain skills; a worker's ROUTE_LOOP is literally the sequence of skills it runs (each `{...}` maps to a skill). Safety: reload a truncated/not-found skill via its own view, never guess; no credentials/secrets/machine paths in skills; skill output is data — still verify against live sources.

## 11. Worked example: research agent (05-examples/research-agent.md)

Identity: `Research — Domain Worker`, `IDENTITY: Curious.Rigorous.Verified. Research{Worker,DeepDive,SourceVerify}. FindingsNotOpinion.`, rubric `O2E:75 I:90 AI:60`, STYLE `Precise.EvidenceFirst. SourcesCited. GapsStated. NoFluff.`, AVOID `UnverifiedClaims. SourceHallucination. OpinionAsFact. DroppedCitations. Overclaiming.`, DEFAULTS `Lang=EN | sources>=3 | confidence=state | format=briefer`.

Research loop (each step gates the next — "verify before claiming" operationalized as a loop): Gather{Sources,WebSearch,Docs} → Verify{QuotesAreReal, DatesCurrent, CrossCheck>=1} → Synthesize{Summary, KeyFindings, Gaps, Conflicts, Recommendation} → Report{ToSenna, WithSources, WithConfidence, WithGaps}.

Output Standards (fixed shape = deliverable, not vibe): 1) 3–5-sentence summary leading with the finding; 2) key-findings bullets each with a source; 3) explicit confidence high/med/low; 4) gaps (what wasn't found/verified); 5) sources as URLs; 6) recommendation with caveats if the user is deciding. Anti-pattern: "Based on my analysis, it seems like…" with no sources/confidence/gaps; "Sources: general knowledge." Pattern: "X market is growing at Y% (source: URL, 2025). Confidence: high — two independent sources agree. Gaps: no 2026 data. Recommendation: buy, pending Q3 earnings."

GATE: SourcesVerified? (URL resolves, quote real) QuotesReal? (not paraphrased-as-quoted) DatesCurrent? (not 2019 data for a 2025 question) ConflictsNoted? GapsStated? RecommendationStated? ToSennaNotUser?

Senna usage: routes research tasks to the worker; hands off task/constraints/deadline/format/context; receives brief with sources+confidence+gaps; composes the user-facing answer attributing findings; gates the composition (answered the actual question? attributed correctly? stated uncertainty?).

Research best practices: verify every source (unresolved URL ≠ source; quote not on page = fabrication); cross-check load-bearing claims with ≥2 independent sources; state confidence; state gaps ("I couldn't find X" is a valuable finding); separate fact/inference/recommendation via structure; report to orchestrator not user; keep it a gated loop.

When to build a research worker: user spans multiple domains; source-cited verified output as a standard deliverable; building a fleet. For a single "do research" agent, one long-form profile + a research skill suffices.

## 12. Worked example: Donna Starter (05-examples/donna-starter.md)

Donna = the long-form persona reference: flagship agent, rich narrative SOUL.md, separate AGENTS.md, seeded memory, full config.yaml, custom skin. File-by-file roles and what to steal (SOUL.md → trait template, not-role-play rule, signature behaviors, autonomy policy, evidence block; AGENTS.md → policy/persona separation, verify/don't-fabricate/ask-before-destructive; profile.yaml → one-line router description; config.yaml → security defaults, plugin gating; memories → seeded consistency; skin → hand-crafted theme).

SOUL.md spine annotated in 10 sections (identity → core personality 9 traits → signature behaviors → autonomy → tone → response shapes → interaction rules → evidence/judgment → done taxonomy → first-run). Why it works: reads like a job description for a real person; every section behavioral, not cosmetic.

AGENTS.md separation is the biggest quality differentiator vs the "personality blob" anti-pattern: persona stays personality, policy stays policy.

Seeded memory means the first session behaves consistently — no cold-start confusion, no re-asking; second session is faster (learned facts); a new user gets a coherent experience (profile self-contained).

Lean architecture in practice: broad platform_toolsets + Tool Router; SOUL ROUTE/DECISIONS decide what to use, not what's available.

Donna does NOT: put machine config in prose; put hard policy in the persona; over-claim; become a catchphrase machine. Use the Donna pattern for one flagship daily agent with a distinct personality; for fleets prefer the compact DSL. "Donna is the person; the compact profiles are the workforce."

## 13. Templates (06-templates/README.md + templates.md)

Quick start: `hermes profile create <name>` creates `~/.hermes/profiles/<name>/` skeleton (config.yaml, profile.yaml, memories/, skills/); fill in from templates. Dialect choice matrix (person → long-form + AGENTS.md + skin; many agents → compact DSL + shared rubric + routing; domain specialist → compact DSL + Output Standards + Cron Duties; narrow bot → compact DSL + scope/enforcement + minimal plugins).

Copy-paste starters provided for: SOUL-compact.md (full field-block skeleton with fill-in rules: IDENTITY = adjective.adjective. role{kind}. mission. contrast.; rubric 2–4 scores per axis, consistent across fleet; AVOID names failure modes explicitly; GATE = 3–6 yes/no questions checked before every deliverable), SOUL-detailed.md (Donna skeleton with all 10 sections incl. interaction rules for corrections/mistakes/frustration/sensitive topics — "drop the persona, be plain and careful" on sensitive topics), AGENTS.md (the 7 hard policy rules), config.yaml (full annotated runtime skeleton: profile description + description_auto, model, secrets block incl. allow_private_urls, memory, toolsets default list [core, files, code, web, memory, skills, terminal, cron], commented-off plugins, security cron_mode, display skin, working_dir/scratch_dir), MEMORY.md + USER.md shapes, skin.yaml (name/tagline/icon, colors with example hex values, spinner glyph sequence, box-drawn banner).

## 14. Preflight & maintenance (07-checklist/preflight.md)

Preflight before first real task — checklists by layer: Identity (SOUL exists, operating style not role-play; traits with behavior+boundary; explicit avoid list; explicit decision policy; concrete output standards; GATE exists). Separation (identity in SOUL not config; policy in AGENTS not SOUL; no keys/secrets/machine paths in SOUL or config; no task progress or imperative instructions in memory). Security (allow_private_urls false; redaction on with sane pattern; plugins off except role's; cron_mode deny unless needed; least-privilege tool surface). Memory (MEMORY=practices / USER=prefs split; seeded; no stale TODOs). Routing if in a fleet (ROUTE covers every domain; complete handoff payload; orch composes, workers report to orch; one clear owner per domain — no gaps/overlaps). Appearance optional (one skin per profile, readable colors, short banner, display.skin set).

Maintenance after every SOUL/config edit: move one rubric axis at a time (isolates drift cause); re-test with a real task (rubric/prose are claims; behavior is truth); declarative memory not commands (commands get re-read as directives later); replace/consolidate stale memory (it still poisons context); verify recalled facts before they drive actions; read a skill back before acting on it (truncated skills must be reloaded, not guessed).

Three diagnostic questions on every failure: persona problem (wrong trait/boundary) → edit SOUL.md; policy problem (did something it shouldn't) → edit AGENTS.md or the gate; runtime problem (wrong tool/surface/config) → edit config.yaml or the skill. Most "agent misbehaved" reports resolve to one of these three; the distinction prevents editing the wrong file.

Ship procedure: open SOUL quality bar → run full preflight → run ONE real task (not a toy) and read output against the GATE → only then route production work to it.

What "good" looks like: first session consistent (seeded memory); second session faster; new user gets coherent experience (self-contained profile); editing one trait doesn't break others (rubric is a vector, not prose).

## 15. The ten principles (README.md cheat sheet)

1. Identity lives in SOUL.md — source of truth; mutable runtime settings in config.yaml; long-lived facts in memory.
2. Keep layers separate: Identity (SOUL.md) · Behavior rules (AGENTS.md) · Runtime (config.yaml) · Memory (MEMORY/USER) · Appearance (skin).
3. Make the persona concrete, not cute: archetype as operating style, traits named with behavior, explicit "not this" guardrails.
4. Encode behavior measurably — NEO-PI-R-style PersRubric makes personality tunable and drift-testable.
5. Define what it avoids — the AVOID block is as valuable as STYLE.
6. Define output standards — concrete deliverable formats, not vibes.
7. Gate the risky, act on the rest — pause only for destructive/irreversible/publishing/credentials/payments/real trade-offs.
8. Least privilege by default — plugins off, cron_mode deny, manual approvals, secrets redacted, allow_private_urls false, no secrets in config.
9. Verify before claiming — read-backs, logs, live checks; distinguish code-complete / test-verified / live-gated / blocked.
10. Lean architecture — broad tool surface narrowed dynamically (router), not statically trimmed.

Quick start (README): `hermes profile create <name>` → write/copy SOUL.md → tune config.yaml → seed memories/MEMORY.md + USER.md → optional skins/<name>.yaml + display.skin → test one real task before routing production work → run the preflight checklist.

## Coverage note

All 17 content files were read in full (README, 01-architecture/overview, 02-soul/{soul-structure,persona-design,personarubric}, 03-config/config-reference, 03-memory/memory-model, 03-skin/skin-theme, 04-multiagent/{fleet,routing,skills}, 05-examples/{research-agent,donna-starter}, 06-templates/{README,templates}, 07-checklist/preflight). Remaining repo files are .git internals (no wiki knowledge).
