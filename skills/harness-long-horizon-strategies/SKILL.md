---
name: harness-long-horizon-strategies
description: Use when task exceeds context window — choose between compaction, structured note-taking, or sub-agent architecture.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a task is going to accumulate more tokens than the context window
can hold — multi-week migrations, deep research tasks, multi-hour coding
sessions, anything where the agent is expected to remember earlier decisions
across many turns. Fires when someone says "we'll just rely on the long
context window" or when a current run is already failing because the agent
has lost track of earlier decisions. The skill names the three canonical
techniques (compaction, structured note-taking, sub-agents) and tells the
team which to reach for first.

## Preconditions

- The task genuinely exceeds the window. For tasks that fit comfortably,
  these techniques are overhead — say so and stop.
- The agent has the ability to write files (notes), spawn sub-processes
  (sub-agents), or trigger summarization (compaction) — at least one of the
  three. Without write access, only compaction is available.
- The team can articulate what counts as "lost context" for this task —
  forgotten decisions, repeated work, broken cross-references. Without a
  failure mode, the technique cannot be tuned.

## Execution Workflow

1. Diagnose the task shape. Conversational back-and-forth (chat with a
   user, exploration with frequent reformulations) tilts toward compaction.
   Iterative work with milestones (migration, refactor, multi-feature
   build) tilts toward structured note-taking. Complex parallel research
   (compare ten papers, audit twenty services) tilts toward sub-agents.
2. Start with the simplest technique that fits. Compaction is the first
   lever — it is built into many agents and needs no new infrastructure.
   Tune it for recall first (capture every load-bearing decision), then
   for precision (drop the verbose tool outputs).
3. If compaction loses critical context across resets, graduate to
   structured note-taking. The agent maintains a NOTES.md or todo list
   that survives between context resets. Define what goes in the file —
   architectural decisions, unresolved bugs, milestones hit, open
   questions — so the agent does not write logs at random.
4. If a single agent still cannot keep up — for example, the research
   workload is genuinely parallel — graduate to sub-agents. Spawn a
   sub-agent per isolated subtask with a clean context, let it use
   10000-plus tokens exploring, and have it return a 1000-to-2000-token
   condensed report. Main agent coordinates.
5. Combining techniques is allowed but should follow the same start-simple
   discipline. A common stable shape: main agent uses note-taking for
   coordination, spawns sub-agents for deep dives, and runs compaction as
   a last-resort safety net before the window fills.
6. Audit the choice after the first long run. If the agent thrashed
   between techniques, simplify. If one technique dominated, consider
   committing to it and dropping the others.
7. Re-check when the model upgrades. Newer models with less context
   anxiety may make compaction enough where note-taking was previously
   required. Long-horizon strategy is not a permanent decision.

## Rules: Do

- Pick the technique by task shape, not by what is fashionable. Sub-agents
  are powerful and over-applied; many teams need only note-taking.
- Tune compaction for recall first. Aggressive compaction that drops
  context the agent later needs is worse than no compaction.
- Define the schema for note-taking explicitly — what categories of
  information go where in NOTES.md. A free-form note file becomes a
  log nobody trusts.
- Give sub-agents a clear, narrow charter and a return-format spec. A
  sub-agent that returns 8000 tokens of unfiltered notes defeats the
  point.
- Combine techniques only after the simplest one demonstrably fails.
  Premature combination is the most common failure mode here.

## Rules: Don't

- Don't assume a larger context window will solve the long-horizon
  problem. Pollution and relevance degrade inside the window too.
- Don't use sub-agents for tasks a single agent could finish in one
  session. The orchestration overhead and handoff loss are not free.
- Don't let compaction strip tool calls or messages that are still
  load-bearing for the current decision — schema the compaction prompt,
  do not trust defaults.
- Don't store agent notes in chat history alone. Chat-only notes vanish
  on context reset, which is exactly the failure mode this skill exists
  to prevent.

## Expected Behavior

After applying the skill, the team has a named long-horizon technique per
task type, a concrete schema or prompt that implements it, and a story
for what the agent does when the window fills. The agent stops "losing
the plot" mid-task. Cross-session continuity becomes mechanical rather
than dependent on the human re-explaining context every time.

When a new long-running task surfaces, the team has a default to reach
for and a checklist of when to graduate. Decisions about which
technique to use stop being argued from first principles each time.

## Quality Gates

- The chosen technique is named and justified in the task plan.
- For compaction: the compaction prompt is committed and tuned for the
  task; recall is verified on a sample reset.
- For note-taking: the schema for what goes in NOTES.md exists in the
  agent's prompt; the file is checked into the repo (or persistent
  storage) per task.
- For sub-agents: each sub-agent has a written charter and a return-
  format spec; the main agent's coordination plan is explicit.
- Combinations are documented and only added after the simpler base
  technique was tried first.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-architecture` (Planner/Generator/Evaluator
is the fixed-role cousin of the dynamic sub-agent pattern) and
`harness-context-rot-budget` (compaction tuning rides the same attention-
budget logic). Methodology phase: 40 (Execução) — long-horizon strategy
is chosen and applied during the run, not at upfront design time.

## Output Artifacts

- A long-horizon-strategy note in the task plan or exec-plan file naming
  the technique and the trigger thresholds.
- Compaction prompts, NOTES.md schemas, or sub-agent charters as
  applicable, committed to the repo.
- Run reports that include where the technique fired and what was
  preserved versus dropped.

## Example Constraint Language

- Use "must" for: naming the technique before the task starts, tuning
  compaction for recall first, schema-ing NOTES.md when note-taking is
  in play.
- Use "should" for: starting with the simplest technique that fits,
  re-auditing the choice after the first long run.
- Use "may" for: combining techniques, swapping techniques mid-task when
  the failure mode shifts.

## Troubleshooting

- **"Compaction keeps dropping the decision we needed"**: the compaction
  prompt is tuned for precision over recall. Rewrite it to preserve all
  architectural decisions, unresolved bugs, and open questions verbatim,
  even if that makes the summary longer.
- **"NOTES.md became a log nobody reads"**: the schema is missing or
  too loose. Restrict the file to a small set of named sections
  (decisions, open questions, milestones) and have the agent write only
  to those sections.
- **"Sub-agents return huge unfiltered dumps"**: the return-format spec
  is missing. Add a hard token cap and a structured template (findings,
  evidence, recommendation) to the sub-agent's prompt.
- **"The team wants sub-agents because they sound impressive"**: this is
  the fashion case. Run the task with note-taking first; if note-taking
  hits the bar, sub-agents are unnecessary complexity.

## Concrete Example

A team starts a multi-week migration — moving 60 services off a legacy
ORM. Hours one through three, the agent uses default compaction at 80%
context and the work goes smoothly. Hour four, the agent starts
forgetting which services have already been migrated and re-proposes
work that was completed an hour earlier. Compaction is preserving the
recent decisions but losing the milestone history. The team graduates
to NOTES.md: the agent maintains a per-service migration status file
(planned, in progress, done, blocked) that survives across context
resets. Hours four through seven go cleanly. Hour eight, three services
turn out to be tightly coupled and need parallel investigation. The
team spawns sub-agents per service group, each charged with returning a
1500-token report. Main agent reads the three reports, updates NOTES.md,
and resumes coordination. The composite shape — note-taking as the
backbone, sub-agents for parallel deep dives, compaction as the
implicit safety net — carries the team through the rest of the
migration without manual context re-injection.

## Sources

- `[[concepts/context-engineering]]`
- `[[sources/context-engineering-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Effective context
  engineering for AI agents" (Rajasekaran, Dixon, Ryan, Hadfield) via
  Danilo's wiki paraphrase. The three-technique taxonomy and the
  start-simplest discipline are the load-bearing parts of the original
  article carried into harness-pack voice.
