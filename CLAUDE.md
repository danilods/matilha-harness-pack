# matilha-harness-pack

> **You lead. Agents hunt. A harness for building complex projects with AI.**

A Matilha companion pack with 22 skills on harness engineering — agent architecture, context engineering, agent-centric codebase, evaluation discipline, and team operational rituals.

## What this pack is

This pack auto-activates skills when user intent matches harness-engineering questions:

- "How do I structure a multi-agent system for a long-running task?" → `harness-architecture`
- "When should I use a workflow vs an autonomous agent?" → `harness-workflow-vs-agent-decision`
- "My team's PRs keep getting the same review feedback — what now?" → `harness-garbage-collection-day` + `harness-nfrs-as-prompts`
- "I need to write evals for my coding agent" → `harness-eval-graders-taxonomy`, `harness-eval-roadmap-0-to-1`

The pack lives in `skills/` and is auto-discovered by Claude Code, Cursor, Codex, and Gemini CLI when installed as a plugin.

## How to use it

Install via `/plugin install matilha-harness-pack` (Claude Code marketplace) or by cloning the repo and pointing your tool at it.

When a user prompt matches a skill's activation description, the Skill tool invokes it. The skill body provides:

- **When this fires** — what triggers the skill
- **Preconditions** — what must be true
- **Execution Workflow** — what the agent does
- **Rules: Do / Don't** — concrete imperatives
- **Companion Integration** — how this skill complements other Matilha packs
- **Concrete Example** — one realistic scenario
- **Sources** — wiki paraphrase chain + book/talk attribution

## Pack discipline (for contributors)

- **Paraphrase always** — never copy source-book or talk sentences verbatim. Attributed inline quotes are OK ("Lopopolo's axiom: 'Code is free'"). Narrative voice is always paraphrased.
- **3 layers of remove** — source → wiki → skill body.
- **Distinct triggering intent** — every skill has a unique activation description (validator enforces ≤80% word overlap pair-wise).
- **Methodology phase mapping** — every skill body declares which Matilha methodology phases (10/20/30/40/50/70) it applies to.

See `docs/wiki-ingestion-workflow.md` for the full authoring discipline.

## Companion to Matilha core

When `matilha-skills` (Matilha core) is installed, core skills delegate automatically:

- `matilha-plan` (Phase 30 spec authoring) → `harness-architecture`, `harness-workflow-vs-agent-decision`, `harness-aci-tool-design`.
- `matilha-design` (cross-phase) → `harness-context-rot-budget`, `harness-jit-retrieval`.
- `matilha-review` (Phase 50, planned) → `harness-eval-graders-taxonomy`, `harness-capability-vs-regression-evals`, `harness-eval-roadmap-0-to-1`.

If Matilha core is absent, pack skills still work standalone.
