---
name: harness-workflow-vs-agent-decision
description: Use when starting any agentic task — decide workflow (predictable path) vs autonomous agent (dynamic) using decision tree.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use at the start of any task where someone is about to point an LLM at a
problem and the question "should this be a workflow or an agent?" has not
been asked. Fires on phrasing like "let's just give Claude tools and let
it figure it out" or "we'll wrap this in a chain of three calls". Both
may be correct — but the choice should be deliberate, not default.
The skill walks through a decision tree that names the task's properties
and points at one of the agentic patterns (or no agent at all).

## Preconditions

- The task is understood at least at the level of "what comes in, what
  goes out, and what counts as done".
- The team can stand up either form factor — a wrapped workflow or a
  loop-with-tools — without architectural blockers.
- Someone can articulate a stopping condition. Open-ended problems
  without a stopping condition default to runaway cost.
- Latency and cost budgets are at least sketched. Both increase
  significantly when moving from single LLM call to workflow to agent.

## Execution Workflow

1. State the task in one sentence. If the sentence reads "classify X and
   route to Y" or "transform A into B in N steps", you are likely in
   workflow territory. If it reads "investigate why X is broken" or
   "build the right thing for Y", you are likely in agent territory.
2. Run the decision tree:
   - Q1: Does the task have binary success criteria + predictable steps?
     → YES: Proceed to Q2 (Workflow territory)
     → NO: Proceed to Q3 (Agent territory)
   - Q2: How do steps decompose?
     → Sequential, fixed → Prompt Chaining
     → Categorized input → Routing (see `harness-routing-parallelization`)
     → Independent parallel → Parallelization (see `harness-routing-parallelization`)
     → Refinement loop helps → Evaluator-Optimizer (see `harness-evaluator-optimizer-loop`)
   - Q3: Can sub-tasks be decomposed dynamically?
     → YES: Orchestrator-Workers (see `harness-orchestrator-workers`)
     → NO: Open-ended exploration → Agent (with tools + stopping condition)
4. Sanity-check by asking: would a single LLM call plus retrieval solve
   this? Many problems do not need agentic systems at all. Agents trade
   latency and cost for performance on hard tasks; if the task is not
   hard in the relevant sense, the trade is bad.
5. If choosing an agent, name the stopping condition (max iterations,
   max cost, observable terminal state) and the sandbox boundary
   (filesystem scope, network access, allowed tool set) before the loop
   runs.
6. Start with the simplest pattern that fits. Add complexity only when
   the simpler version demonstrably underperforms — measured, not
   suspected.
7. Document the choice and its rationale in a short note alongside the
   implementation. Future maintainers (human and agent) need to know
   why this is a workflow and not an agent, or vice versa.

## Rules: Do

- Pick deliberately between workflow and agent. The choice is not a
  preference — it has measurable cost, latency, and reliability
  consequences.
- Default to the simpler pattern. Single LLM call < workflow < agent in
  cost, latency, and risk.
- Name the stopping condition before launching any agent loop. Cost
  runaway happens fastest when no one wrote the terminal state down.
- Choose framework vs raw API based on production maturity. Frameworks
  accelerate prototyping but obscure internals; raw API suits production.
- Re-evaluate the choice when the task evolves. A task that started
  predictable may have grown open-ended.

## Rules: Don't

- Don't reach for an agent because it sounds modern. If a workflow fits,
  a workflow ships faster, costs less, and breaks less.
- Don't use a workflow when the task genuinely needs dynamic
  decomposition (the orchestrator-workers and agent patterns exist
  because some problems require them).
- Don't skip the "is this even an agentic problem?" check. Some tasks
  are best solved by a single LLM call plus retrieval, or by traditional
  code with no LLM.
- Don't run an agent without sandbox boundaries and a stopping condition.
  Compounding errors and unbounded cost are the failure modes.

## Expected Behavior

After applying the skill, the team has a named pattern for the task
(prompt chaining, routing, parallelization, orchestrator-workers,
evaluator-optimizer, agent, or "no agentic system needed") and a written
rationale. The chosen pattern matches the task's actual properties
rather than the team's preferred form factor.

Conversations about implementation move past "should this be an agent?"
quickly because the question has already been answered explicitly.

## Quality Gates

- The chosen pattern is named in writing alongside the implementation.
- The rationale references the task properties (predictability,
  branching, independence, refinement value, openness).
- For agents: the stopping condition and sandbox boundary are explicit.
- The simplest fitting pattern was chosen unless evidence supports more
  complexity.
- The choice is revisited if the task evolves.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Hands off to `harness-routing-parallelization`, `harness-orchestrator-workers`,
or `harness-evaluator-optimizer-loop` once a workflow pattern is selected;
hands off to `harness-architecture` when the choice is "long-running agent
system". Methodology phase: 30 (Skills/Agents) — this is the first decision in
that phase.

## Output Artifacts

- Short rationale note (3-10 lines) committed alongside the
  implementation, naming the chosen pattern and its triggering task
  properties.
- For agents: a stopping-condition + sandbox-boundary spec.

## Example Constraint Language

- Use "must" for: explicitly choosing a pattern, naming stopping
  conditions for agents, sandboxing agent tool access.
- Use "should" for: starting with the simpler fitting pattern, raw API
  over framework when production matures, documenting the rationale.
- Use "may" for: adopting a framework for prototyping speed, mixing
  patterns when the task has heterogeneous segments.

## Troubleshooting

- **"The task evolved and our workflow can't handle the new shape"**:
  the choice needs revisiting. Re-run the decision tree. Often the right
  move is orchestrator-workers (dynamic decomposition) or an upgrade to
  evaluator-optimizer (iterative refinement).
- **"We picked an agent and cost is unbounded"**: the stopping condition
  was missing or too loose. Add max iterations, max cost, and explicit
  terminal-state checks.
- **"The agent gets stuck in a loop"**: agents need ground truth from
  the environment at each step (tool results, code execution). If the
  environment isn't returning useful signals, the agent has no anchor.
  Either improve tool feedback or move to a workflow.
- **"Workflow is too rigid for the variety of inputs we see"**: the
  task probably needs routing as a front door, then specialized
  workflows per category. See `harness-routing-parallelization`.

## Concrete Example

Two tasks land in the same week. Task A: classify support tickets and
route to a specialist queue (billing, technical, account). The decision
tree fires: binary success criterion (right queue or wrong queue),
categorized input, classification step needed. Pattern: routing. A
small classifier (Haiku) plus three specialist handlers ships in two
days. Task B: investigate why CI is flaky on the new microservice. The
tree fires differently: open-ended exploration, sub-tasks unpredictable,
ground truth comes from log inspection, test re-runs, and code reads.
Pattern: agent (with tool access to logs, test runner, and source).
The team gives it a stopping condition (max ten iterations or "root
cause identified with linked evidence") and a sandbox (read-only on
prod logs, full read on the repo, no writes). Task A runs as a tight
predictable workflow; Task B runs as an open agent loop. Picking the
wrong form for either would have been visibly worse.

## Sources

- `[[concepts/agentic-patterns]]`
- `[[sources/building-effective-agents-anthropic]]`
- Synthesized from Erik S. and Barry Zhang's 2026 Anthropic engineering
  post "Building Effective AI Agents" via Danilo's wiki paraphrase. The
  decision tree extends the article's pattern catalog with explicit
  question ordering.
