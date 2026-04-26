---
name: matilha-harness-trigger
description: "Use when the user mentions agent architecture, multi-agent, orchestrator, planner, executor, subagent, eval, evaluation, LLM pipeline, prompt engineering, RAG, retrieval-augmented generation, grader, transcript, context window, harness, or any topic related to building AI agent systems and LLM-powered workflows. Fires independently of compose to ensure matilha-harness-pack skills activate whenever agentic AI domain appears."
category: harness-pack
version: "1.0.0"
---

## When this fires

User prompt mentions any keyword from matilha-harness-pack's domain: agent architecture, multi-agent, orchestrator, planner, executor, subagent, eval, evaluation, LLM pipeline, prompt engineering, RAG, retrieval-augmented generation, grader, transcript, context window, harness.

This skill fires independently of `matilha:matilha-compose` and `CLAUDE.md` activation rules — its keyword-rich description is the activation surface for the Skill tool's matcher. It guarantees that whenever the user prompt touches the agentic AI / LLM-harness domain, at least one matilha-harness-pack skill enters the conversation.

## Execution Workflow

1. **Pack presence check.** Inspect the ambient skill list for skills in the `matilha-harness-pack:*` namespace. Examples: `matilha-harness-pack:harness-architecture`, `matilha-harness-pack:harness-orchestrator-workers`, `matilha-harness-pack:harness-eval-roadmap-0-to-1`.

2. **Pack installed path.** If ≥1 skill from `matilha-harness-pack` is visible:
   - Identify the user's sub-intent within the harness domain (designing an orchestrator/worker layout, picking eval graders, sizing context window budget, choosing workflow vs autonomous agent, structuring AGENTS.md, etc.).
   - Emit a compact domain acknowledgment (≤2 lines — no full sigil, that belongs to compose). Example: `AI harness domain detected. Pulling matilha-harness-pack guidance for <sub-intent>.`
   - Invoke the most relevant pack skill via the Skill tool. Prefer one specific skill over listing many.

3. **Pack not installed path.** If no `matilha-harness-pack` skills are visible:
   - Emit: "AI Harness skills not installed. Run `/matilha-install` and select `harness` to add 22 harness skills."
   - Proceed with default flow (`matilha:matilha-compose` if visible, otherwise `superpowers:brainstorming`).

## Rules

- Do NOT emit the full compose sigil. The sigil belongs to compose; this skill emits a compact pack-specific acknowledgment.
- Do NOT block on pack absence — emit the nudge and continue with the default flow.
- Prefer invoking a specific pack skill over listing all skills.
- This trigger is a routing surface, not a craft skill — hand off to the chosen pack skill quickly.

## Why this exists

Wave 5h adds deterministic activation surfaces to compensate for probabilistic Skill-tool matching. Compose's routing table covers the central path; this trigger covers prompts that bypass compose (e.g., when CLAUDE.md is absent, or when a domain keyword appears mid-conversation without a compose-classifiable phase). Together they form the Maximum Activation guarantee for matilha-harness-pack.
