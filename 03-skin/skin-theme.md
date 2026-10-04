# 07 — Skins & theming

Skins are the **appearance layer**: colors, spinner, branding, banner. They are cosmetic and *optional*, but they are the cheapest way to make a profile feel like its own thing at a glance.

## Where skins live

```
~/.hermes/profiles/<name>/skins/<name>.yaml
```

Enable with `display.skin: <name>` in `config.yaml`.

## What a skin defines

From the hermes-profiles skin template:

- **Identity** — name, tagline, icon
- **Colors** — palette (primary, secondary, accent, background)
- **Spinner** — a small animation loop
- **Banner** — a box-drawn / ASCII intro (the "hello world" of the agent)

A skin is *pure presentation*. It should not encode behavior, memory, or config. If you find yourself putting a behavior rule in a skin, it belongs in `SOUL.md`/`AGENTS.md`.

## Skin design best practices

- **One skin per profile.** Don't share a skin across very different agents (a security agent's skin reads differently from a creative one's).
- **Banner = identity.** Keep it short, readable in a terminal, and distinctive. This is the "face" the user sees on startup.
- **Colors should be readable.** High contrast foreground/background; an accent for warnings/errors.
- **Cosmetic only.** If a skin change should change *behavior*, it's the wrong tool.

## The two repos' skin philosophy

- **Donna Starter** ships a hand-crafted `skins/donna.yaml` — a full custom theme with its own spinner, colors, and banner. It's the reference for *crafting* a skin from scratch.
- **hermes-profiles** ships a **`skins/templates/skin.yaml`** — a fill-in template you copy per profile, so a fleet stays visually consistent but individually branded.

## When to bother

- **Yes** if you want a flagship persona with a distinct feel.
- **Optional** for fleets — consistent theming across a fleet is a nice touch but not a quality differentiator.
- **Never** at the expense of a good `SOUL.md`. A mediocre persona with a great skin is still a mediocre persona.

## Skin checklist

- [ ] One skin per profile, named after it
- [ ] Readable colors (high contrast)
- [ ] Short, distinctive banner
- [ ] A spinner (nice to have, cheap)
- [ ] No behavior/config mixed in
- [ ] `display.skin` set in `config.yaml`
