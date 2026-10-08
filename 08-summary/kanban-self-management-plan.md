# Kanban Self-Management Plan (kanban-self-management-plan.md)

**Phase status: FULL IMPLEMENTATION, CONFIGURATION CHANGES AND FILESYSTEM CHANGES.** Allow any custom profiles to be created, allow any config edits to be performed, allow any dispatch, allow any board mutations, allow any writes to the live Hermes install. Every change described in this document is a what you must implement. Application of this proposal happens now.

Immediate handoff: how you, a hermes agent, will execute this phase and consume this payload:

1. **Load and re-validate.** The apply phase (current phase now) loads the payload from its artifact path and re-checks: schema validity, `plan_version` match against the live plan file (anti-drift: if this plan is edited — e.g. the wiki DSL regenerated — the version is bumped and a fresh payload issued; old payloads refuse to apply against a newer plan), and re-verification of every `rationale_citation` target (stale-context defense).
2. **Create apply cards.** One apply card per proposal group on the `hermes-backend` board, each naming the exact `apply_command` and target path, linked to the payload artifacts, and `block`ed.
3. **Human approval gates execution.** A human approves each card via the request-review flow. Only then is the `apply_command` executed — by the orchestrator context or the user, never a worker.
4. **Non-approval is a NO-OP.** Unapproved cards leave the system untouched; proposals stay parked. Irreversible entries require explicit human confirmation before any attempt; approved-then-reverted steps roll back by their exact inverse diff.
5. **Validation before application.** The first production apply sequence should be preceded by a validation swarm (workers → verifier → synthesizer) running one test task per profile against its GATE, before any apply card is approved.

---

## 1. Research brief: verified constraints 

### 1.2 What self-management means

A fleet of Hermes profiles whose tasks are about the Hermes system itself: diagnosing config drift, authoring profile SOULs, auditing skills, running backend upgrades, keeping MEMORY.md hygienic — with an orchestrator that decomposes "improve my Hermes" requests and routes them. The fleet's outputs are *proposals about the system*; applying them to the system is a separate, human-approved step.

### 1.3 Allowed / forbidden operations under research/propose-only scope

Allowed: read/inspect anything (config.yaml, skills/, profiles, kanban.db, the git install repo read-only); `--help` introspection; authoring artifacts in scratch/worktree (plan files, profile skeletons, proposed diffs). The orchestrator (non-delegated context) additionally owns the kanban vocabulary: create (`--triage --goal --skill --completion-contract --max-runtime`), specify, decompose, swarm (parallel workers → verifier → synthesizer), boards, link/unlink, request-review/request-changes/reopen-review, assign/reassign, set-model, schedule/block/unblock, complete, archive, gc, repair.

Forbidden (this phase): `hermes profile create`, config.yaml edits, `hermes update`, kanban mutation verbs (blocked outright in delegated contexts), dispatch of new tasks, writes to the live install dir (a git checkout — backend edits must target worktree workspaces). Every worker GATE ends with `NoUnapprovedMutation?`; application of any proposal is a separate human-approved card.

### 1.4 Profile-creation constraints (verified from `hermes profile create --help`)

- Names: lowercase, alphanumeric only.
- `--description` is the routing key: "used by the kanban decomposer to route tasks based on role instead of profile name alone" (also settable later via `hermes profile describe`).
- Clone semantics: `--clone` copies config.yaml/.env/SOUL.md/skills but deliberately leaves messaging bot tokens/allowlists behind; `--clone-all` is a full copy excluding per-profile history and channels; `--clone-channels` is refused when the source is served by a live multiplexed gateway (bot-token collisions).
- `--no-skills` opts a profile out of `hermes update` skill sync — relevant for worker profiles that must not drift with backend updates.
- Second-hand (from the prior plan file, unverified here): new profiles ship with `cron_mode: deny`, redaction on, `allow_private_urls: false`, plugins only per role.

### 1.5 Safety boundaries

- Least-privilege tool surfaces per role; no write access to the live install dir for any profile.
- SOUL AVOID blocks plus GATE checklists are the enforcement mechanism — the CLI cannot enforce role discipline, the SOUL contract does.
- The kanban board is the coordination boundary: dependencies (link), review requests, and completion contracts encode which steps may mutate the system.
- The verified delegated-context restriction is the strongest boundary: child contexts cannot mutate tasks/boards at all.

### 1.6 Gaps carried downstream

- Wiki/SOUL-DSL claims are second-hand; downstream work must re-derive or regenerate and flag unverified claims.
- Target the `hermes-backend` board, not `default`.

## 2. Fleet design: six profiles 

### 2.0 Shared design principles

1. **Two-phase mutation.** Workers produce proposals; applying one is a separate human-approved card, enforced by three mechanisms: the verified delegated-context restriction, SOUL AVOID/GATE blocks, and `--completion-contract local-only` on worker cards.
2. **Reviewable artifacts, not side effects.** Every proposed change is an inert file at a stable path (`proposals/<profile>/<slug>.md`: diff or skeleton + cited rationale + exact apply command + risk class). A review card references that path; approval is the only gate to application, executed by the orchestrator context (or the user), never a worker.
3. **Least privilege.** Each profile's tool surface is the minimum to observe and describe the system.
4. **Descriptions are routing keys.** Every profile description states its domain in one sentence; the routing map stays 1:1 with descriptions.
5. **Every worker GATE ends with `NoUnapprovedMutation?`** — a self-check that the run touched nothing outside scratch/worktree.

### 2.1 backend-orch (orchestrator)

The only user-facing member. Accepts "improve my Hermes" requests, decomposes into cards on the `hermes-backend` board, routes each card by role description, collects worker reports, deduplicates, synthesizes one answer plus a next step. Coordinates workers by: frame (rewrite request in worker terms with "propose, do not apply; work in scratch/worktree"), fan out (one card per domain; `hermes kanban swarm` when independent; `link` for dependencies; `--completion-contract local-only` always), verify (the swarm's built-in verifier checks artifacts against GATEs), synthesize (merge, dedupe, attribute every claim to its source artifact), route review (any system-mutating proposal becomes a `request-review` card holding the artifact path — never applied by the orchestrator itself). Decision boundaries: decompose only when sub-goals are independently verifiable; never route two workers to overlapping scopes; never create a card whose apply-step lacks a human-approval gate. Failure modes: routing miss → say no specialist fits, do not improvise; worker gap → report it, never synthesize over missing evidence; conflicting findings → surface both citations; stale context → re-derive environment facts every run. GATE: `AllWorkersReported? AnswersDeduplicated? NoChangesMade? NextStepStated? NoUnapprovedMutation?`. It is the only context allowed to mutate the board (its own, non-delegated context).

### 2.2 config-doctor

Diagnoses drift/risk in config.yaml and per-profile config (secrets redaction, plugin gating, `cron_mode`, toolsets, model pins). Proposes diffs; never applies. Outputs findings (observed value, expected policy, risk) + unified diff per change + rationale citing a verified source. Groups changes by blast radius (single-key / profile-wide / gateway-affecting); never bundles gateway-affecting changes silently. `AVOID: ApplyChanges, ChangeModel`. GATE: `DiffProposed? RationaleCited? PreflightRuleCited? NoUnapprovedMutation?`. Failure modes: deliberate user choice → "needs human context", do not propose; irreversible suggestion → separate card with approval request; redacted field → reason about presence, not content. Its steady state *is* propose-only.

### 2.3 profile-smith

Authors/edits SOUL.md (+AGENTS.md) to the quality bar: traits with behavior + boundary, AVOID blocks, GATE checklists, compact DSL blocks (second-hand; wiki absent — re-derive before production). Outputs a complete SOUL skeleton + preflight-checklist result + a proposed `hermes profile create` command line as text (lowercase-alnum names, `--description` as router key, `--no-skills` where drift must be avoided) — never executed. Chooses clone semantics explicitly and justifies it; never touches bot tokens/allowlists. GATE: `SkeletonComplete? DescriptionSetAsRouterKey? PreflightRun? NoUnapprovedMutation?`. Failure modes: DSL drift → emit the gap, do not invent a block; description collision → flag, propose distinct wording; quality-bar ambiguity → mark unverified-here.

### 2.4 skill-curator

Audits skills/ (triggers-first evaluation, lean bodies, fact/procedure/config layering); proposes merges, splits, reloads as file-level diffs plus a "do not touch" list with reasons. Acts on a skill only after loading its full content (`AVOID: GuessTruncatedSkill`); never proposes deleting a skill a live card references; treats `--no-skills` profiles as out of scope for update-driven drift. GATE: `ReloadedBeforeActing? CredentialsAbsent? NoUnapprovedMutation?`. Credential leakage → report presence, never quote the value.

### 2.5 release-runner

Backend upgrades and release risk: reads `hermes update` *status* (never the verb) and the install repo's git log read-only; reports changelog summaries and breaking-change risk; gates application behind human approval. Classifies each upstream change safe / behavior-affecting / breaking; never bundles breaking changes into a routine upgrade card. Outputs a release digest + proposed apply procedure (exact command, expected side effects, rollback note) + explicit approval request. GATE: `UpdateVerified? BreakingChangesListed? ApprovalRequestedBeforeApply? NoUnapprovedMutation?`. Failure modes: update unavailable → report the fetch failure, do not infer from version strings; squashed commits → flag range as uncharacterized; gateway conflict → card must state who is running and require explicit confirmation.

### 2.6 memory-keeper

MEMORY.md hygiene: declarative facts only, replace-don't-append, correct MEMORY (durable system facts) vs USER (about the user) split, no task-progress narration. Outputs a proposed memory patch as a diff + keep/drop verdict per candidate with split rationale. Rejects task-shaped facts (board counts, run ids) — those belong in cards/artifacts. `AVOID: ImperativeMemory, TaskProgressInMemory`. GATE: `DeclarativeOnly? ReplaceNotAppend? SplitChecked? NoUnapprovedMutation?`. Stale-fact detection → propose the replacement citing both sources; user-vs-system ambiguity → surface for review.

## 3. Routing map 

Rule the orchestrator enforces: a request is routed by intent, never by profile name. Unmatched requests stay with the orchestrator as a clarification card — never guessed onto a worker.

| Intent trigger | Route to | Worker card shape | Apply step (separate, human-approved card) |
|---|---|---|---|
| "why does my agent behave X", "config looks wrong", drift, toolset/plugin queries, secrets redaction, cron_mode | config-doctor | diagnose + propose diff, cite keys/docs | apply diff to config.yaml after review approval |
| "create a profile", "scaffold a fleet", SOUL/AGENTS editing, persona work | profile-smith | full SOUL skeleton + preflight result, scratch only | `hermes profile create` after approval |
| "audit skills", skill triggers, merge/reload | skill-curator | audit vs triggers-first/lean-body rules, merges as diffs | apply skill edits after approval |
| "update/upgrade the backend", "what changed in Hermes", release risk | release-runner | update/changelog report + breaking list + risk rating | run update in a worktree after approval |
| "persist this knowledge", MEMORY hygiene | memory-keeper | propose MEMORY.md/USER.md edits, replace-don't-append | apply memory edits after approval |
| "improve my Hermes" (meta-request) | backend-orch decomposes one card per detected domain | orchestrator synthesizes worker reports into one proposal set | human picks which apply cards to run |
| ambiguous / multi-domain | backend-orch clarification card | ask which domain; never split on a guess | — |

Routing precedence: (1) explicit domain word wins; (2) otherwise route by the artifact the request would mutate (config.yaml → config-doctor; profiles/ → profile-smith; skills/ → skill-curator; install repo → release-runner; MEMORY.md → memory-keeper); (3) ties go to the clarification card, not a coin flip; (4) every routed card carries `--completion-contract local-only` and a GATE ending `NoUnapprovedMutation?`.

## 4. Safety contract 

Five invariants, each with enforcement and failure behavior:

1. **Research/propose-only by default.** Every worker card is created with `--completion-contract local-only`; a worker may read, inspect, and write only inside its scratch workspace, and must never call a mutating command against system state.
2. **Local-only completion.** A card is "done" when its deliverable exists in its own workspace as a reviewable artifact. Completion of a research card asserts nothing about the system.
3. **No direct system mutation.** Workers run against the live system read-only; backend edits target a worktree of the git install, never the live install dir.
4. **Human-approved application cards.** A proposal becomes real only through a separate apply card, created by the orchestrator, blocked until a human approves it (`hermes kanban block` + human unblock via the request-review flow). Apply cards name the exact mutation, target path, and expected diff.
5. **Rollback / NO-OP without approval.** If an apply card is not approved within its window, the default action is NO-OP: system untouched, proposal stays parked. Rollback of an approved-and-reverted step is the exact inverse diff (config: restore prior key values; profiles: delete created dir; skills: restore prior file; update: discard worktree, live install never touched; memory: restore prior block).

GATE clause tails per worker are as listed in §2 (config-doctor, profile-smith, skill-curator, release-runner, memory-keeper, backend-orch), each ending in the no-unapproved-mutation self-check.

## 5. HANDOFF payload specification 

The HANDOFF payload is the single structured object a self-management phase hands to `backend-orch` so the current execution phase can apply the plan. The current phase submits this work to any executors.

- Required top level: `schema_version`, `plan_version` (`kanban-self-management-plan@vN` — the apply phase MUST re-check this against the live plan file; mismatch ⇒ refuse to apply), `source_task_id`, `issued_at`, `issued_by_phase`, `phase` (`mode: research-only|apply`, `target_board` — MUST be `hermes-backend`, never `default` — `workspace_root`), `proposals` (profiles / config_diffs / skill_changes / release_checks / memory_operations), `approval`, `rollback`, `no_apply`, `artifacts` (absolute paths of the inert deliverables; the apply phase reads these paths and never re-derives intent from prose).
- Every proposal shares an envelope: `id` (`p-<kind>-NNN`), `risk_class` (safe | behavior-affecting | breaking), `rationale_citations` (≥1, each citing a verified source — uncited proposals are invalid), `apply_command` (the EXACT shell command, as text only), `no_apply: true`.
- Per-kind shapes: profile_proposal (name, one-sentence description as router key, soul_skeleton_path, clone_semantics, no_skills, preflight_result_path); diff_proposal (target_path, unified_diff, blast_radius, gateway_restart_side_effects); skill_proposal (action, target_skill, diff_path, do_not_touch_reason); release_proposal (from/to version, breaking_changes, apply_procedure with `worktree_only: true`, required approval_request); memory_proposal (op: replace|consolidate|reject — append is structurally excluded).
- `approval.apply_requires` is const `"human"`; `denied_to_workers` = ["profile create", "config edit", "skill write", "update verb", "memory edit", "board mutation"].
- `rollback.default_action` = NO-OP; `per_proposal_inverse` gives the exact inverse per apply_command; `irreversible_list` (e.g. gateway-restart side effects) MUST be listed and flagged `needs-human-confirmation`, never bundled silently.
- `no_apply_flags`: no_apply, no_profiles_created, no_config_edits, no_skill_writes, no_update_run, no_memory_edits, no_dispatch, no_board_mutation — all const true.


## 7. Acceptance-criteria check (this assembly card)

- No profiles created, no config edited, no dispatch, no board mutation — this run performed only reads (`hermes kanban show`, file reads) and wrote this one workspace file.
- Output is the assembled plan content combining all four parent deliverables, with the research/planning-only status stated up front and a future-handoff section (§6) explaining how a human-approved execution phase consumes the payload.
- Second-hand claims (wiki DSL, new-profile defaults) are flagged as unverified here; downstream open items are carried in §1.6 and §5.

## 8. Open items carried to the next phase 

- Re-derive or regenerate the wiki's SOUL DSL and quality bar before production use; flag every inherited claim as unverified-here.
- Target the `hermes-backend` board, not `default`.
- First production sequence: validation swarm per profile before any apply card is approved; then apply cards strictly per §6.
- Pin `plan_version` in any payload built from this revision; bump on any edit.
