# matilha-harness-pack

> **You lead. Agents hunt. A harness for building complex projects with AI.**

A Matilha companion pack with **22 harness-engineering skills** synthesized from Anthropic engineering posts, OpenAI's Codex agent-centric blog, and Ryan Lopopolo's "Harness Engineering" talk (AI Engineer London 2026).

This pack codifies discipline for teams running AI-assisted development:

- **Agent architecture** — Planner/Generator/Evaluator + sprint contracts.
- **Agentic patterns** — workflow vs agent, routing, parallelization, orchestrator-workers, evaluator-optimizer, ACI design.
- **Context engineering** — context rot, JIT retrieval, long-horizon strategies.
- **Agent evaluation** — graders taxonomy, capability vs regression, 9-step roadmap.
- **Agent-centric codebase** — AGENTS.md as index, docs/ as system of record, Ralph Wiggum loop.
- **NFRs as prompts** (Lopopolo) — lint messages with remediation, test source-code structure, review agents by persona, Garbage Collection Day ritual.
- **Foundational axiom** — "Code is free" (Lopopolo): how the scarcity lens shifts in token-billionaire economics.

## Install

### Claude Code (via Matilha marketplace)

```
/plugin install matilha-harness-pack
```

Or locally during development:

```
/plugin install /path/to/matilha-harness-pack
```

### Cursor

Same `/plugin install` flow — compatible with Claude Code's `.claude-plugin/plugin.json` format.

### Codex CLI

Point at the repo; enable `multi_agent = true` in `~/.codex/config.toml` if you want subagent dispatch.

### Gemini CLI

Install as an extension; `gemini-extension.json` lands in a follow-up release (Claude Code + Cursor + Codex ready out of the box).

## Skills (22 total)

**Harness architecture (2):** `harness-architecture`, `harness-evolution`.

**Foundational axiom (1):** `harness-code-is-free`.

**NFRs as prompts (5):** `harness-nfrs-as-prompts`, `harness-lint-as-prompt`, `harness-test-source-code-structure`, `harness-review-agents-by-persona`, `harness-garbage-collection-day`.

**Agentic patterns (5):** `harness-workflow-vs-agent-decision`, `harness-routing-parallelization`, `harness-orchestrator-workers`, `harness-evaluator-optimizer-loop`, `harness-aci-tool-design`.

**Agent-centric codebase (3):** `harness-agents-md-as-index`, `harness-docs-as-system-of-record`, `harness-ralph-wiggum-loop`.

**Context engineering (3):** `harness-context-rot-budget`, `harness-jit-retrieval`, `harness-long-horizon-strategies`.

**Agent evaluation (3):** `harness-eval-graders-taxonomy`, `harness-capability-vs-regression-evals`, `harness-eval-roadmap-0-to-1`.

Each skill is derived from 1-3 wiki concept pages plus source attribution. See individual `skills/<slug>/SKILL.md` files.

## Integration with Matilha core

When `matilha-skills` (Matilha core) is installed, core skills delegate automatically:

- `matilha-plan` (Phase 30 spec authoring) → `harness-architecture`, `harness-workflow-vs-agent-decision`, `harness-aci-tool-design`, `harness-orchestrator-workers`.
- `matilha-design` (cross-phase) → `harness-context-rot-budget`, `harness-jit-retrieval`.
- `matilha-hunt` (Phase 40 execution) → `harness-ralph-wiggum-loop`, `harness-long-horizon-strategies`.
- `matilha-review` (Phase 50 quality, Wave 3c planned) → `harness-eval-graders-taxonomy`, `harness-capability-vs-regression-evals`, `harness-eval-roadmap-0-to-1`.

If Matilha core is absent, pack skills still work standalone.

## Overlap with matilha-ux-pack

Two skills derive from material that ux-pack also uses. They cover a DIFFERENT triggering intent:

| harness-pack skill | ux-pack sibling | Distinction |
|---|---|---|
| `harness-eval-roadmap-0-to-1` | `ux-swiss-cheese-errors` | Agent-eval defense layers (Reason's Swiss Cheese applied to LLM evals) vs human-error tolerance design |
| `harness-context-rot-budget` | `cog-cognitive-load` | LLM attention budget (token economics, context rot) vs human working memory (Miller's 4±1) |

The other 20 skills declare "No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational." Install all three packs for complementary coverage.

## Attribution

Skills synthesize principles from:

- Anthropic Engineering: "Harness Design" (Prithvi Rajasekaran), "Building Effective Agents", "Context Engineering for AI Agents", "Demystifying Evals" (2025-2026).
- OpenAI: "An Agent-Centric World" Codex blog (2026), "Harness Engineering: How to Build Software When Humans Steer, Agents Execute" (Ryan Lopopolo, AI Engineer London 2026).
- General agent literature, ACI design (Anthropic), and operational practices observed in production teams running 24/7 agentic development.

Skill content is Danilo's original synthesis via Obsidian wiki paraphrase. Source posts/talks are credited but not quoted verbatim except as inline attributed citations.

See `docs/wiki-ingestion-workflow.md` for the authoring discipline.

## Contribute

Pack is open source. To propose a new skill or refine one:

1. `docs/wiki-ingestion-workflow.md` in this repo.
2. `matilha-skills/docs/matilha/skill-authoring-guide.md` (upstream style guide).
3. `matilha-skills/docs/matilha/pack-authors.md` (upstream pack-author contract).

## License

MIT.
