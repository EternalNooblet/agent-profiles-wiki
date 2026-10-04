# 05 — `config.yaml` reference & best practices

`config.yaml` is the **runtime layer**. It is *never* the place for identity or persona — those live in `SOUL.md`. But it's where most "why does this profile behave oddly?" problems are solved.

## Security & least privilege (the top tier)

The two source repos are aggressive about defaults. This is the most important block in the file.

```yaml
secrets:
  redaction: true
  redaction_exclusions: []
  secret_pattern: "password|secret|token|key|api_key"
  secret_redaction: "REDACTED"
```

- **`allow_private_urls: false`** — block localhost/internal endpoints from web tools.
- **Plugins default OFF.** Only enable what the profile actually needs:
  - Research profile → `search, memory, skill`
  - Infra/homelab profile → `terminal, git, shell`
  - A moderation bot (e.g., `gamehub-mod`) → `discord` and **nothing else**
- **`cron_mode: deny` by default.** Autonomous recurring jobs must be *opt-in*, and only enabled where a profile genuinely needs them (research, infra, homelab, gamehub-mod).
- **No secrets in `config.yaml`** — never commit provider keys.

## Model & routing

```yaml
model:
  provider: custom
  model: your-model-id
```

- **Lean architecture = broad tool surface, dynamic narrowing.** Register a wide tool surface and narrow *per request*. Do not statically trim tools to make the profile "look smaller."
- Use a **Tool Router / routing layer** for multi-tool fleets — it narrows the deterministic first turn without deleting capability.
- `max_concurrent_children` — limit parallel subagents per profile. Research fleets run 4; narrow agents run 1–2.
- **Routing by profile:** use the `profile` param to dispatch to a specialized agent instead of one monolith doing everything.

## Memory

```yaml
memory:
  provider: null          # or 'mem0', 'chromadb', 'pgvector'
  file: memory.md         # persistent memory file path
```

- File memory is the default (the two `memories/*.md` files).
- Vector providers (mem0, chromadb, pgvector) give semantic recall — enable only when you have enough history to benefit and a way to verify recall quality.

## Toolsets & permissions

```yaml
toolsets:
  default: [core, files, code, web, browser, memory, skills, terminal, cron]
  platform_toolsets: [...]
```

- **Scope by role.** A moderation bot should not have terminal. A research agent should have web + memory + skill, not browser automation.
- `default_agent: agent` and `platform_toolsets` define what the *system* can call; the SOUL's `ROUTE`/`DECISIONS` define what *this persona* decides to use.

## Display / theming

```yaml
display:
  skin: dave              # or 'senna', 'magnus' — from skins/
  show_full_transcripts: true
```

The skin is cosmetic (colors, spinner, banner) — see [07 — Skins](03-skin/skin-theme.md).

## Working directory & scratch

```yaml
working_dir: ~/projects
scratch_dir: /tmp/scratch   # ephemeral, pruned 24h
```

- `working_dir` = persistent user workspace.
- `scratch_dir` = ephemeral for probes/builds (never write permanent state here; on Windows it maps to the hermes scratch dir).
- Keep these **per-profile** so agents don't collide.

## The three "do not" rules

1. **Do not put identity/persona in `config.yaml`.**
2. **Do not put runtime behavior in `SOUL.md`.** (No provider keys, no timeouts, no plugin toggles in prose.)
3. **Do not commit secrets.** `config.yaml` is local-only.

## Config best-practices summary

| Concern | Best practice | Source pattern |
|---|---|---|
| Security | `allow_private_urls: false`, redaction on, no secrets in file | Donna `config.yaml` |
| Plugins | Default off; enable only what the role needs | hermes-profiles `getting-started` |
| Cron | `deny` by default; opt-in per profile | hermes-profiles profiles |
| Tools | Broad surface + dynamic narrowing (router) | both repos |
| Memory | File by default; vector when scale demands + verified | both repos |
| Display | Skin = cosmetics only | hermes-profiles `skins` |

See [Templates](06-templates/README.md) for a complete, annotated `config.yaml` you can start from.
