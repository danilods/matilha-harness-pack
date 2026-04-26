# Changelog

## [0.3.0] — 2026-04-26 — Wave 5h: deterministic trigger skill

### Added

- **`matilha-harness-trigger` skill** — independent activation surface for agent/harness domain. Keyword-rich description (orchestrator, planner, executor, subagent, multi-agent, eval, grader, RAG, retrieval, transcript, context window, harness, agent architecture, prompt, LLM, etc.) ensures pack skills enter the conversation whenever the domain appears in user prompts.
- Complements `matilha-skills`'s routing table (`skills/matilha-compose/routing-table.md`); together they form Wave 5h's Maximum Deterministic Activation surface.

### Notes

- Fully additive: existing 22 skills untouched. Pack continues to work standalone or with `matilha-skills` (Matilha core).
- Trigger is a routing surface, not a craft skill — emits a compact domain acknowledgment and hands off to the most relevant pack skill via the Skill tool.
- No behavior change when pack is uninstalled: trigger emits a `/matilha-install` nudge and yields to default flow.

## [0.1.0] — 2026-04-20

First release. 22 harness-engineering skills synthesized from:

- **Anthropic Engineering** — "Harness Design" (Prithvi Rajasekaran), "Building Effective Agents", "Context Engineering for AI Agents", "Demystifying Evals" (2025-2026).
- **OpenAI** — "An Agent-Centric World" Codex blog (2026), "Harness Engineering: How to Build Software When Humans Steer, Agents Execute" (Ryan Lopopolo, AI Engineer London 2026).

### Skill clusters

- **Harness architecture (2):** `harness-architecture`, `harness-evolution`.
- **Foundational axiom (1):** `harness-code-is-free`.
- **NFRs as prompts (5):** `harness-nfrs-as-prompts`, `harness-lint-as-prompt`, `harness-test-source-code-structure`, `harness-review-agents-by-persona`, `harness-garbage-collection-day`.
- **Agentic patterns (5):** `harness-workflow-vs-agent-decision`, `harness-routing-parallelization`, `harness-orchestrator-workers`, `harness-evaluator-optimizer-loop`, `harness-aci-tool-design`.
- **Agent-centric codebase (3):** `harness-agents-md-as-index`, `harness-docs-as-system-of-record`, `harness-ralph-wiggum-loop`.
- **Context engineering (3):** `harness-context-rot-budget`, `harness-jit-retrieval`, `harness-long-horizon-strategies`.
- **Agent evaluation (3):** `harness-eval-graders-taxonomy`, `harness-capability-vs-regression-evals`, `harness-eval-roadmap-0-to-1`.

### Overlap discipline

Two skills declare cross-pack overlap with `matilha-ux-pack`:

- `harness-eval-roadmap-0-to-1` complements `ux-swiss-cheese-errors` (agent-eval defense vs human-error tolerance).
- `harness-context-rot-budget` complements `cog-cognitive-load` (LLM attention budget vs human working memory).

### Validation

22 skills × 7 validation checks each + 6 plugin/uniqueness/overlap = 160 new tests in `matilha` CLI registry suite. Full suite: **903 tests passing** (743 baseline + 160 harness-pack).

### Authoring discipline

3 layers of remove (source → wiki → skill), attributed inline quotes only, methodology phase mapping in body. See `docs/wiki-ingestion-workflow.md`.
