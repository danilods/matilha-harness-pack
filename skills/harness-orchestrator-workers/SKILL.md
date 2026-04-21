---
name: harness-orchestrator-workers
description: Use when sub-tasks are unpredictable a priori — orchestrator decomposes dynamically, delegates to workers, synthesizes results.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a task can't be cleanly pre-decomposed because the necessary
sub-tasks depend on the input itself. Fires on phrasing like "we don't
know which files this change will touch until we look at the codebase"
or "the right next step depends on what we find in the logs". This is
the canonical case for orchestrator-workers: the orchestrator inspects
the input, decides at runtime what sub-tasks are needed, dispatches them
to workers, and synthesizes the results. Distinct from routing
(pre-defined handlers) and from parallelization sectioning (pre-known
splits).

## Preconditions

- The team has decided this is a workflow problem, not an autonomous
  agent (see `harness-workflow-vs-agent-decision`).
- Routing and parallelization sectioning have been ruled out — the
  decomposition genuinely depends on the input.
- Workers can be spawned as separate calls (or subagents) with their
  own context. An orchestrator that holds all worker context in one
  process is just a single agent.
- The synthesis step is feasible — the orchestrator (or a dedicated
  synthesizer) can combine worker outputs into a coherent result.

## Execution Workflow

1. Frame the task to the orchestrator with three things: the input, the
   tools it has for inspection, and the protocol for spawning workers.
   The orchestrator's first move is always inspect-then-decompose.
2. The orchestrator inspects the input — reads code, queries logs,
   classifies the request — and decides at runtime: how many workers,
   what each worker does, what context each needs. This decomposition
   is the load-bearing step.
3. Workers run with focused context. Each worker gets only the slice of
   the problem the orchestrator assigned, plus the tools it needs.
   Worker context windows stay manageable because the orchestrator
   fanned out the work.
4. The orchestrator (or a separate synthesis step) collects worker
   outputs and combines them. For coding tasks, this often means
   reviewing each worker's diff and producing a single coherent change
   set. For investigation tasks, it means correlating findings into a
   root-cause narrative.
5. Iterate if needed. The orchestrator may discover, after seeing
   worker outputs, that a follow-up sub-task is necessary. The pattern
   supports multiple rounds — but each round should be justified by
   what the prior round revealed, not by uncertainty about the original
   plan.
6. Bound the orchestration. Set a max number of worker calls per
   orchestration cycle and a max number of cycles. Without bounds,
   orchestrator-workers can drift into agent-like cost runaway.
7. Capture the orchestrator's decomposition reasoning in the output.
   Future maintainers (and future orchestration runs) benefit from
   seeing why a particular fan-out shape was chosen.

## Rules: Do

- Use orchestrator-workers when the sub-tasks are genuinely unpredictable
  a priori. If you can pre-decompose, use parallelization sectioning
  instead — it's simpler and cheaper.
- Keep workers context-focused. The point of fanning out is to give
  each worker a smaller, sharper context than the orchestrator would
  have if it did the work alone.
- Make the synthesis step explicit. Worker outputs that pile up without
  a designed combine step produce noise, not insight.
- Bound the orchestration. Max worker count per cycle, max cycles per
  task, max total cost — all written down before launch.
- Log the orchestrator's decomposition rationale. The decomposition is
  the load-bearing decision and deserves audit.

## Rules: Don't

- Don't use orchestrator-workers when sub-tasks are predictable. The
  added complexity costs latency and tokens for no benefit.
- Don't let workers spawn their own workers. That's an agent loop, not
  orchestrator-workers — the supervision shape is different.
- Don't skip the synthesis step. Raw worker outputs are inputs to
  synthesis, not the answer.
- Don't run unbounded. Orchestrator-workers without limits is one
  prompt away from an autonomous agent with worse observability.

## Expected Behavior

After applying the skill, the team has a workflow that handles inputs
whose decomposition cannot be pre-specified. The orchestrator inspects,
decides, dispatches, and synthesizes. Workers stay context-focused.
Total cost and latency are bounded explicitly. The output includes both
the synthesized result and a record of how the work was decomposed —
so future runs (and humans reviewing them) can see the pattern.

For coding and investigation tasks especially, this pattern unlocks
work that would otherwise require either a single overburdened agent
(slow, lossy) or a pre-decomposition that the team cannot actually
write down (because they don't know the shape until they look).

## Quality Gates

- The decomposition is genuinely runtime-dependent (a pre-decomposition
  was attempted and rejected with reason).
- Workers run with focused context — not the full orchestrator
  context.
- The synthesis step is named and produces a coherent combined output.
- Bounds (max workers per cycle, max cycles, max cost) are written
  down and enforced.
- The orchestrator's decomposition rationale appears in the output.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Sits next to `harness-routing-parallelization` (the pre-decomposed
alternatives) and `harness-evaluator-optimizer-loop` (which can wrap
individual workers when their output benefits from refinement).
Methodology phase: 30 (Skills/Agents) and 40 (Execução), where orchestration
runs happen.

## Output Artifacts

- The orchestrator prompt with explicit inspect-then-decompose protocol.
- Worker prompts (one per worker type, parameterized by orchestrator).
- Synthesis prompt or code that combines worker outputs.
- The combined output file plus a decomposition-rationale section.

## Example Constraint Language

- Use "must" for: bounded worker count and cycle count, focused worker
  context, explicit synthesis step.
- Use "should" for: capturing decomposition rationale in output,
  iterating only when prior round justifies it.
- Use "may" for: parameterizing worker prompts (one worker definition,
  many invocations) versus distinct worker prompts per role.

## Troubleshooting

- **"The orchestrator keeps spawning workers but no progress is made"**:
  the decomposition is failing. Either the inspection step isn't
  surfacing useful structure, or the orchestrator's prompt doesn't
  give it enough scope to commit to a decomposition. Inspect the
  orchestrator's reasoning trace.
- **"Worker outputs are inconsistent and synthesis produces noise"**:
  workers got too little shared context. They each made locally
  reasonable choices that don't combine. Add a small shared-context
  block (NFRs, conventions, glossary) to every worker prompt.
- **"Cost is unbounded"**: missing or loose bounds. Add hard caps on
  worker count per cycle and cycles per task. Re-run.
- **"It feels like we should just use an agent"**: maybe so. The
  difference is supervision: orchestrator-workers spawns workers
  explicitly with bounded scope; an agent spawns its own subtasks via
  tool calls in a loop. If the team needs the latter's flexibility and
  can accept the cost, switch.

## Concrete Example

Bug report: "login is broken after the deploy this morning". A
pre-decomposition isn't possible — the team doesn't know yet whether
the cause is in code, infra, data, or config. The orchestrator inspects
the report (timestamp, affected user count, error rate spike from
observability) and decomposes at runtime into three workers: worker A
reads the last twenty commits and surfaces anything touching auth,
session, or middleware; worker B queries observability for the error
fingerprint and correlates with deploy timing; worker C reproduces the
issue locally with a fresh user account and instruments the failing
request. Each worker has a focused context (commits only, logs only,
local repro only) and produces a structured finding. The orchestrator's
synthesis step correlates: commit X (worker A) introduced a race in
session middleware that observability shows manifesting at deploy time
(worker B) and that local repro confirms reliably under simultaneous
login attempts (worker C). Total: three worker calls, one synthesis
call, one cycle. A single agent investigating the same bug would have
held all three contexts in one window and likely thrashed; pre-
decomposition would have required the team to know the answer before
asking the question.

## Sources

- `[[concepts/agentic-patterns]]` (section Orchestrator-Workers)
- `[[sources/building-effective-agents-anthropic]]`
- Synthesized from Erik S. and Barry Zhang's 2026 Anthropic engineering
  post "Building Effective AI Agents" via Danilo's wiki paraphrase.
  The bounded-orchestration discipline extends the source's pattern
  description with operational guardrails learned from coding-agent
  use cases.
