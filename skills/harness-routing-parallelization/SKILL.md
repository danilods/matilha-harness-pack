---
name: harness-routing-parallelization
description: Use when input falls into distinct categories or sub-tasks can run simultaneously — routing for cost/specialization, parallelization for diversity.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team has chosen a workflow (see `harness-workflow-vs-agent-decision`)
and the workload either splits into clear input categories that benefit from
specialized handlers, or decomposes into independent sub-tasks that can run
in parallel. Fires on phrasing like "all queries go through one big prompt
that handles everything" (routing candidate) or "we run this sequentially
even though the steps don't depend on each other" (parallelization
candidate). The skill names the right variant — routing for cost or
specialization, parallelization sectioning for independent work,
parallelization voting for diversity of opinion.

## Preconditions

- The task has been classified as workflow, not agent (otherwise see
  `harness-orchestrator-workers` or the agent pattern).
- For routing: the categories are stable enough that a classifier can
  identify them with high accuracy, and at least two categories
  meaningfully benefit from different handling.
- For parallelization: sub-tasks must be genuinely independent (no
  cross-dependencies), or the same task repeated for diversity.
- Output aggregation is feasible — for parallelization, the team knows
  how the parallel results will be combined or chosen between.

## Execution Workflow

1. Identify whether the task fits routing, parallelization sectioning,
   or parallelization voting. Routing: distinct input categories,
   different handlers. Sectioning: one task that splits into independent
   sub-tasks. Voting: one task run N times for diverse perspectives.
2. For routing: list the categories. For each, decide whether the
   handler should be a different model (cost routing — Haiku for
   classification, Opus for deep work) or a different prompt (specialization
   routing — billing prompt, technical prompt, account prompt). Cost and
   specialization can compose.
3. Build the classifier. An LLM classifier is fine when categories are
   semantic; a rules-based classifier is faster and cheaper when the
   signal is structural. Measure classifier accuracy on a representative
   sample before relying on it — misrouted inputs degrade specialist
   performance silently.
4. For parallelization sectioning: split the work explicitly. Each
   sub-task has a defined input slice and an independent output. Run
   the sub-tasks concurrently. Aggregate results in a separate step
   that merges or stitches outputs.
5. For parallelization voting: run the same task with N different prompts
   (different criteria, different framings) or N seeds. Aggregate via
   majority vote, ensemble synthesis, or guardrail-style "any failure
   blocks" — the aggregation rule depends on what the votes are for.
6. Use voting deliberately for guardrails. One agent processes the user
   query; a second agent screens the output for policy violations. Two
   independent passes catch what one pass would have missed.
7. Measure and re-tune. Routing accuracy, parallelization speed-up
   versus token cost, and voting diversity all need observation. Patterns
   that fit well early can drift as input distribution shifts.

## Rules: Do

- Choose routing when input categories are stable and benefit from
  specialized handling — lower cost (cheaper model on easy queries),
  better quality (specialist prompt on hard queries), or both.
- Choose parallelization sectioning when sub-tasks are genuinely
  independent and the latency win pays for the orchestration overhead.
- Choose parallelization voting when one perspective is insufficient and
  diversity adds measurable value — code review with multiple criteria,
  guardrail screening, ensemble decision-making.
- Measure classifier accuracy before trusting routing in production.
- Aggregate parallel outputs in an explicit, named step — never as an
  afterthought.

## Rules: Don't

- Don't route when the categories are unstable or the classifier is
  noisy. Misrouted inputs degrade specialist performance and the team
  blames the specialists.
- Don't section sub-tasks that have hidden dependencies. Race conditions
  in agent workflows are as real as in concurrent code.
- Don't vote when one prompt is sufficient. N-way voting multiplies cost;
  the diversity must justify it.
- Don't conflate routing with orchestrator-workers. Routing picks a
  pre-defined handler from a fixed set; orchestrator-workers decomposes
  dynamically into worker calls that did not exist a priori.

## Expected Behavior

After applying the skill, the team has either a routing front door
(classifier plus specialists) or a parallel decomposition (sectioned
sub-tasks or voted ensemble) appropriate to the task. Cost drops on
routed workflows because trivial inputs no longer hit the expensive
model. Latency drops on sectioned workflows because independent work
no longer queues. Quality improves on voted workflows because diverse
perspectives surface what a single pass missed.

## Quality Gates

- For routing: classifier accuracy is measured on a representative
  sample and meets a stated threshold before going live.
- For sectioning: each sub-task has a defined input slice and an
  independent output; the aggregation step is explicit.
- For voting: the diversity source (different prompts, different seeds,
  different models) and the aggregation rule are both named.
- Cost and latency are measured before-and-after; a refactor that
  doesn't improve either is suspect.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Often follows `harness-workflow-vs-agent-decision` (which chose workflow) and
may precede `harness-evaluator-optimizer-loop` (which adds refinement on top
of a chosen handler). Hands off to `harness-orchestrator-workers` if the team
realizes the sub-tasks were unpredictable rather than independent.
Methodology phases: 30 (Skills/Agents design) and 40 (Execução, where the
parallel runs happen).

## Output Artifacts

- Routing: classifier prompt or rules, plus per-category handler
  prompts, all committed.
- Sectioning: a decomposition spec naming each sub-task's input slice
  and output, plus the aggregation step.
- Voting: the N prompts (or seeds) and the aggregation rule, plus the
  rationale for why diversity matters here.

## Example Constraint Language

- Use "must" for: measured classifier accuracy before routing in
  production, named aggregation step for parallelization, explicit
  diversity source for voting.
- Use "should" for: cost routing (cheap model first, escalate on hard
  cases), guardrail voting (one processor, one screener) for
  user-facing systems.
- Use "may" for: composing routing with parallelization (route to a
  specialist that itself runs sectioning internally).

## Troubleshooting

- **"Specialists are underperforming"**: check classifier accuracy
  first. Misrouted input is the most common cause of "the
  technical-queries handler is bad". Re-measure on a representative
  sample.
- **"Parallel sectioning didn't speed things up"**: hidden dependency
  or aggregation overhead. Profile the actual concurrent run; if one
  sub-task waits on another's output, it wasn't independent.
- **"Voting produces contradictory outputs we can't aggregate"**: the
  aggregation rule wasn't designed for the diversity source. Majority
  vote works for classification; for synthesis tasks, use an explicit
  meta-prompt that reads all N outputs and produces the merged answer.
- **"Routing scope keeps growing — we keep adding categories"**: the
  task may have outgrown routing. If categories are no longer stable,
  consider orchestrator-workers (dynamic decomposition) instead.

## Concrete Example

Customer support routing: incoming tickets are classified into billing,
technical, and account categories. Billing routes to a cheap classifier
(Haiku) plus a templated response handler — most billing queries are
"why was I charged?" with a deterministic answer. Technical routes to
a specialist prompt (Opus) loaded with the product's troubleshooting
runbook plus tool access to log search. Account routes to a hybrid:
classifier plus a human escalation when account changes are involved.
Cost per ticket drops sixty percent versus the prior single-prompt
setup; resolution quality on technical tickets measurably improves
because the specialist has the runbook in context.

Separately, code review parallelization: every PR fans out to three
voting prompts — security (looking for injection, auth bypass,
secrets), performance (N+1 queries, blocking calls, allocation
hotspots), and style (naming, structure, NFR adherence). The
aggregation step compiles findings by file, deduplicates near-identical
flags, and posts a single review comment. One pass with one generic
"review this code" prompt would have surfaced roughly a third of these
findings.

## Sources

- `[[concepts/agentic-patterns]]` (sections Routing and Parallelization)
- `[[sources/building-effective-agents-anthropic]]`
- Synthesized from Erik S. and Barry Zhang's 2026 Anthropic engineering
  post "Building Effective AI Agents" via Danilo's wiki paraphrase.
  Cost-routing and guardrail-voting framings extend the source's
  pattern descriptions with operational rules of thumb.
