# Wiki Ingestion Workflow — harness-pack edition

> How to synthesize a Matilha harness-pack skill from the Obsidian wiki.

## Why this exists

The `matilha-harness-pack` skills are derived from Danilo's Obsidian wiki, which is itself a curated paraphrase of Anthropic engineering posts, OpenAI's Codex blog, and Ryan Lopopolo's AI Engineer London 2026 talk. This doc codifies the discipline so that:

1. Future pack authors (including other Matilha packs) reproduce the same discipline.
2. Attribution is consistent — always 3 layers of remove from source posts/talks, with attributed inline quotes only when authoring requires it.
3. Activation is reliable — every skill has a distinct triggering intent.
4. Skill style follows `matilha-skills/docs/matilha/skill-authoring-guide.md`.

## The 5-step process per skill

### 1. Select triggering intent

From the pack's skill inventory (in the spec or `README.md`), pick one row. The row's activation description is the contract — the Skill tool matches this against user intent.

Example: `harness-architecture` → "Use when designing multi-agent system for long-running tasks — Planner/Generator/Evaluator + sprint contract."

### 2. Load wiki sources

Read the wiki concept page + 1-2 source pages listed in the skill's source mapping. Note:

- Key principles (what the skill is teaching).
- Concrete examples from the wiki or source posts (which you'll rephrase).
- Citations back to the original posts/talks.

Example sources for `harness-architecture`:
- `~/Documents/Memory/Memory/wiki/concepts/harness-engineering.md`
- `~/Documents/Memory/Memory/wiki/sources/harness-design-anthropic.md`
- `~/Documents/Memory/Memory/wiki/sources/harness-engineering-lopopolo-openai.md` (operational lens)

### 3. Paraphrase + synthesize

Draft the skill body in Matilha's voice. Principles:

- **Paraphrase always** — do not copy source-post sentences. Do not copy wiki-page sentences verbatim either. Aim for 3 layers of remove.
- **Attributed inline quotes are OK** — Lopopolo's punchy axioms ("Code is free", "token billionaire", "Continue typed = harness failure") may appear as inline attributed quotes ("Lopopolo's axiom: 'Code is free'") but never as Matilha narrative voice.
- **Include ONE concrete example** per principle (e.g., "When a long-running task hits 80% context window, the orchestrator spawns a sub-agent with a 200-token handoff artifact rather than compacting in-place").
- **Cite inline with wikilinks**: "(see `[[concepts/harness-engineering]]` for the full Planner/Generator/Evaluator treatment)".
- **Close the body with a Methodology phase mapping**: every skill body declares "Methodology phase: 30 (Skills/Agents)" or similar, pointing to the Matilha methodology phase(s) where this skill applies.

### 4. Validate against the style guide

Check:

- [ ] Frontmatter matches `matilha-skills/docs/matilha/skill-authoring-guide.md` strict schema: `name`, `description` starting with "Use when" or "When", `category: harness`, `version: "1.0.0"`, `requires: []`, `optional_companions: []`.
- [ ] Body has all 12 required sections (When this fires, Preconditions, Execution Workflow, Rules: Do, Rules: Don't, Expected Behavior, Quality Gates, Companion Integration, Output Artifacts, Example Constraint Language, Troubleshooting, Concrete Example) + mandatory `## Sources` section (13th).
- [ ] Mandatory `## Sources` section: 1-3 wikilinks in `[[wiki-path]]` format + post/talk attribution note.
- [ ] Body length 170-220 lines (validator allows 150-300).

### 5. Cross-check activation uniqueness + overlap disclosure

`grep -r "^description: " skills/*/SKILL.md` and visually scan. Also cross-check against `matilha-ux-pack` and `matilha-growth-pack` skills — this pack has 2 intentional overlap points:

- `harness-eval-roadmap-0-to-1` overlaps with `ux-swiss-cheese-errors` (same model, different application). Phrase: "Complements matilha-ux-pack:ux-swiss-cheese-errors at agent-eval-defense vs human-error-tolerance".
- `harness-context-rot-budget` overlaps with `cog-cognitive-load` (same metaphor, different agent). Phrase: "Complements matilha-ux-pack:cog-cognitive-load at LLM-attention-budget vs human-working-memory".

The other 20 skills declare: "No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational."

Each overlap-skill `## Companion Integration` section MUST contain its declared phrase verbatim (validator enforces).

## Example: how `harness-architecture` was authored

Approx 30-minute process:
1. Selected intent: "designing multi-agent system for long-running tasks" (from inventory).
2. Read `concepts/harness-engineering.md` + `sources/harness-design-anthropic.md` + `sources/harness-engineering-lopopolo-openai.md` (15 minutes).
3. Drafted body sections — synthesized Planner/Generator/Evaluator with sprint contract framing; added Lopopolo's operational lens (team rituals) as Companion Integration cross-reference (12 minutes).
4. Added Sources + Methodology phase (30) + validated 13 sections + line count (2 minutes).
5. Cross-checked against other harness skills (`harness-evolution`, `harness-workflow-vs-agent-decision`) — narrowed triggering intent to "designing" (not "evolving" or "deciding pattern") (1 minute).

## When to promote to a meta-skill

If future Matilha packs reveal this workflow as repetitive + formulaic, consider promoting to a `matilha-wiki-ingest` meta-skill. That skill would read a wiki page URL, prompt for triggering intent, and generate a draft SKILL.md body.

Wave 5c continues to ship this as documented discipline.
