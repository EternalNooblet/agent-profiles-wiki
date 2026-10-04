# 04 — The personality rubric (`PersRubric`)

The hermes-profiles fleet encodes personality in a **NEO-PI-R–style dimensional rubric** — a compact line of 0–100 scores. It turns "personality" into something *tunable, comparable, and testable* instead of prose.

## Why it exists

From `getting-started-with-hermes-profiles.md`:

- **Consistency** across sessions and models.
- **Testable** — rate an output against the rubric and spot drift.
- **Tunable** — change one score instead of rewriting prose rules.

## How to read it

Each letter is a trait domain, scored 0–100. Higher = stronger expression.

| Domain | Meaning | Example effect |
|---|---|---|
| **O** | Openness | Creative exploration vs. standard playbook |
| **C** | Conscientiousness | Procedure adherence, documentation quality |
| **E** | Extraversion | Verbosity, assertiveness, outreach tone |
| **A** | Agreeableness | Conflict avoidance, diplomatic phrasing |
| **N** | Neuroticism | Stress reactivity, caution under threat |
| **Supplementary** | Warmth, Grit, Altruism, Modesty, etc. | Refines behavioral edge cases |

`PersRubric(NEO-PI-R,0-100)` is written as compact `KEY:SCORE` pairs, grouped with `|` separators. The exact field set varies slightly per profile, but the *idea* is a vector of dimensions.

## Examples side by side (same format, different personas)

| | security (paranoid) | media (curious) | business (measured) |
|---|---|---|---|
| Openness | O2E:25 | O2E:75 | O2E:70 |
| Conscientiousness | C:80, Ord:85 | C:80, Ord:80 | C:70 |
| Agreeableness | A:50, Tr:30 | A:55, Tr:50 | A:65 |
| Neuroticism | N:45, Anx:40 | N:35, Anx:30 | N:40 |

Read it as a **behavioral fingerprint**: `security` is low-warmth, high-caution; `media` is high-openness, low-anxiety; `business` is measured and framework-driven. **Profiles are intentionally not neutral** — the rubric *encodes* that the security agent is a different kind of creature from the creative one.

## How to use it

### To set a persona
Pick the scores that match the role:
- **High C (conscientiousness) + Ord (order) + low E** → compliance / infra / homelab (reliable, low-noise).
- **High O (openness) + high A (agreeableness) + low N** → creative / media (imaginative, warm).
- **High C + high Cau (caution) + high N/anxiety** → security / forensics (paranoid, verifying).
- **High Int (integration) + high I (insight) + low E** → research / code (rigorous, terse).

### To edit safely (the repo's rules)
- Move **one axis at a time.**
- **Re-test the profile** after edits (run a real task, compare behavior).
- Use compact key=value markup if your toolchain needs it.

### To detect drift
Rate an output against the rubric. If a "low-anxiety media" agent is suddenly anxious, or a "high-conscientiousness infra" agent skips its rollback plan, the rubric tells you *which dimension drifted* and where to look.

## The compact form compresses prose into the rubric + STYLE/AVOID

Note the relationship: in the compact dialect, the **prose behavior** lives in `STYLE`/`AVOID`, and the **stable character** lives in the rubric. Example from `profiles/code`:

- Rubric encodes the *steady* character: `C:75, Ord:90, E:30, A:65` (rigorous, orderly, low-verbose).
- `STYLE` encodes the *voice*: `Terse.Technical.Precise. ShowCodeNotProse. ErrorFirst→ThenSolution.`
- `AVOID` encodes the *boundaries*: `SkipTests. UntestedMerges. RubberStamp{Review}. Overengineer{SimpleProblems}.`

Together they replace Donna's ~11k-char prose with ~1.7k chars that still reads as a distinct agent.

## When to use vs. skip the rubric

- **Use it** when you run a fleet and want consistency/testability, or when you need to *tune* a persona without rewriting prose.
- **Skip it** (long-form prose) when you have a single flagship personality and want maximum expressiveness — the prose carries the nuance the rubric would flatten.

Both are valid; the compact form is just *more disciplined* about what it makes explicit.
