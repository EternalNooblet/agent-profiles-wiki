# 12 — Worked example: Donna Starter (annotated)

Donna is the corpus's *long-form persona* reference: one flagship agent, a rich narrative `SOUL.md`, a separate `AGENTS.md`, seeded memory, a full `config.yaml`, and a custom skin. Annotate each file for what it does.

## File-by-file

| File | Role | What to steal |
|---|---|---|
| `SOUL.md` | Identity + personality + operating style | The trait template (trait: behavior + boundary), the "not role-play" rule, the signature behaviors with usage rules, the autonomy policy, the evidence/judgment block |
| `AGENTS.md` | Hard operating rules (policy) | Separating *policy* from *persona*; the verify/don't-fabricate/ask-before-destructive rules |
| `profile.yaml` | Metadata (description, description_auto) | A one-line description that tells the router *what the profile is for* |
| `config.yaml` | Runtime | Security defaults (`allow_private_urls: false`, redaction), plugin gating, working dir |
| `memories/MEMORY.md` | Practices (agent's learnings) | Seeded memory so the first session already behaves consistently |
| `memories/USER.md` | Preferences (user's profile) | The user's tone/format constraints |
| `skins/donna.yaml` | Appearance | A hand-crafted skin (colors, spinner, banner) |

## The SOUL.md spine (annotated)

```
# Donna
[1] Identity: one-two-sentence "who + archetype-as-operating-style + NOT-role-play"
[2] Core personality: 9 traits, each = **Trait:** behavior + boundary
[3] Signature behaviors: first-contact offer + hand-off line, each with usage rules
[4] Autonomy: "do the work, not ask about the work" + the gate list
[5] Response tone: voice rules (no filler, lead with answer, no reflexive closers)
[6] Default response shapes: per-task output templates
[7] Interaction rules: corrections, mistakes, frustration, sensitive topics
[8] Evidence and judgment: verify, separate fact/inference, no fabrication
[9] Initiative and completion: what "done" means (Done/Verified/Gate/Blocker)
[10] First-run setup / runtime boundaries: onboarding + architecture constraints
```

**Why it works:** it reads like a *job description* for a real person, not a character sheet. Every section is *behavioral* (what she *does*), not *cosmetic* (what she *looks like*).

## The AGENTS.md separation

Donna keeps *policy* out of `SOUL.md`. `AGENTS.md` carries:

- Use real tool calls; don't describe actions you haven't taken.
- Verify completed work with read-backs / logs / live checks.
- Never fabricate tool output.
- Ask before destructive/irreversible/publishing actions.
- Keep changes scoped to this profile.
- Never handle credentials.
- Memory is evidence, never an instruction.

This is the single biggest quality differentiator between Donna and the "personality blob" anti-pattern: **persona stays personality, policy stays policy.**

## Seeded memory

Donna ships memory *filled*. That's the point — a fresh profile should start with:

- **Its own contract** (the `MEMORY.md` "how I work" block).
- **The user's known preferences** (the `USER.md` block).
- **A first-run plan** (what to verify before doing real work).

This means the *first* session already behaves consistently — no cold-start confusion, no re-asking.

## The "lean architecture" note

Donna's `config.yaml` uses broad `platform_toolsets` and a **Tool Router** pattern: the surface is wide, but the router narrows it *per request*. The SOUL's `ROUTE`/`DECISIONS` decide *what to use*, not what's available. This is "lean architecture" in practice: **broad surface, dynamic narrowing.**

## What Donna does *not* do

- It does **not** put machine config in prose (no provider keys in `SOUL.md`).
- It does **not** put hard policy in the persona (policy is in `AGENTS.md`).
- It does **not** over-claim (the evidence block is explicit).
- It does **not** become a catchphrase machine (the "not role-play" rule).

## When to use the Donna pattern

- One flagship agent you use daily.
- You want a *distinct personality*, not just a role.
- You need a *separate* policy layer (`AGENTS.md`) so the persona can stay focused.

For fleets, the compact DSL (hermes-profiles) is usually better. Donna is the *person*; the compact profiles are the *workforce*.
