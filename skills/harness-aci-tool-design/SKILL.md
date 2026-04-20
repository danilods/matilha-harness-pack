---
name: harness-aci-tool-design
description: Use when designing tools for LLM agents — Agent-Computer Interface principles, poka-yoke, format choice, tool description as docstring for junior dev.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team is designing or revising tools that LLM agents will call —
custom MCP tools, function-calling schemas, CLI wrappers, file-edit
primitives. Fires on phrasing like "we'll just expose the API as a tool"
or "the agent should figure out the format from the JSON schema". Both
miss the point: tool design for agents is its own discipline (Agent-
Computer Interface) and Anthropic's reported finding is that ACI
optimization gave more value than prompt optimization in SWE-bench. The
skill walks through the ACI principles and surfaces the most common
failure modes — bad format choice, missing poka-yoke, vague descriptions.

## Preconditions

- The team is designing or revising tools that agents will call (not
  chat-only interactions, not pure prompts).
- A workbench or test harness exists where the team can observe agent
  tool-use behavior across multiple runs. ACI design without
  observation is guesswork.
- Someone owns each tool's spec — descriptions, examples, edge cases —
  not just its implementation.
- The team is willing to iterate. First-draft tools rarely survive
  contact with real agent runs.

## Execution Workflow

1. Choose the tool's input format with the model's training data in
   mind. Markdown, plain text, and YAML beat heavily-escaped JSON for
   anything the model has to write. Reserve strict JSON for what is
   genuinely structured (lookups, IDs) and let prose stay prose.
2. Eliminate formatting overhead the model can't reliably produce.
   Don't require exact line counts, don't require every quote escaped,
   don't require the agent to count brackets. Each formatting tax
   compounds across tool calls.
3. Give the agent room to think before committing. Reasoning tags,
   planning fields, or a separate "scratchpad" parameter let the
   model warm up before writing the load-bearing argument. Without
   this, the agent commits to a constraint with no thinking and
   produces brittle output.
4. Apply poka-yoke. Make errors structurally hard rather than
   politely warned about. Require absolute paths instead of relative
   (no ambiguity after `cd`). Require unique-anchor strings instead
   of line numbers (no off-by-one drift). Reject ambiguous calls at
   the schema layer instead of warning in the description.
5. Write the tool description as a docstring for a junior developer
   — examples, edge cases, format expectations, common mistakes,
   limits. The description is where the agent learns the contract;
   skimping here costs hours of debug later.
6. Test in a workbench. Run the tool against ten realistic agent
   tasks; observe where the agent struggles, misuses parameters, or
   gets stuck. Each observed failure is feedback for the next
   description revision.
7. Iterate. ACI is rarely right on draft one. Treat the tool's
   description and schema as living documents that revise as the
   model and the workload change.

## Rules: Do

- Choose formats close to what the model saw in natural text. Markdown
  and plain prose beat escaped JSON for anything the model writes.
- Apply poka-yoke at the schema layer. Absolute paths, unique anchors,
  reject-on-ambiguity — make wrong calls structurally impossible.
- Write tool descriptions as docstrings for a junior developer.
  Examples, edge cases, format, limits, common mistakes. Verbose
  descriptions pay back on every future call.
- Give the agent thinking room before committing constraints. Planning
  fields, reasoning tags, or a scratchpad parameter.
- Iterate in the workbench. Observe real agent runs, find the
  failures, revise the description and schema.

## Rules: Don't

- Don't require formatting overhead the model can't reliably produce —
  exact line counts, escaped quotes, perfectly balanced brackets.
- Don't write tool descriptions as one-liners. The agent uses the
  description as its primary guide; one-liners produce unreliable
  calls.
- Don't trust the schema alone. Schemas describe shape, not intent.
  The description carries intent.
- Don't assume the first draft is right. ACI design is iterative; the
  first version of any non-trivial tool will reveal failure modes the
  designer didn't anticipate.

## Expected Behavior

After applying the skill, the team's agent tools have descriptions
verbose enough for a junior developer to use correctly, formats close
to natural text, schema-level poka-yoke that prevents the most common
errors, and explicit space for the agent to think before committing
constraints. Agent iteration count drops on tasks that use the tools.
Errors that previously required mid-loop debugging are now caught at
the schema layer.

The team's tool-design practice shifts from "expose the API" to "design
the interface for an LLM caller", with the workbench as the place where
draft tools become production tools.

## Quality Gates

- Each tool's description includes examples, edge cases, format, and
  limits.
- Input format is markdown or natural text where the agent writes
  prose; strict JSON only where structure is genuinely needed.
- Schema-level poka-yoke is in place for the most common failure modes
  (absolute paths, unique anchors, reject-on-ambiguity).
- The tool has been tested in the workbench across at least ten
  realistic tasks and the description revised based on observed
  failures.
- A planning or scratchpad field exists for tools that ask the agent
  to commit to non-trivial constraints.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Often invoked alongside `harness-orchestrator-workers` (workers need
well-designed tools to stay context-focused) and inside `harness-architecture`
(the Generator and Evaluator both need ACI-aware tooling). Methodology phase:
30 (Skills/Agents) — tool design is part of agent design.

## Output Artifacts

- Tool spec per tool: schema + description (with examples, edge cases,
  format, limits) + workbench test cases.
- Workbench observation log: real agent failures and the description
  revisions they triggered.
- Optional: a tool-design checklist the team can apply to new tools.

## Example Constraint Language

- Use "must" for: descriptions with examples and edge cases, format
  matching natural-text training distribution where the agent writes
  prose, schema-level poka-yoke for the most common errors.
- Use "should" for: workbench testing across at least ten realistic
  tasks, planning/scratchpad fields for non-trivial tools.
- Use "may" for: how aggressive the poka-yoke is (some teams prefer
  warnings; others reject — choose based on agent reliability data).

## Troubleshooting

- **"The agent keeps misusing this tool despite a clear schema"**:
  the description is probably under-spec. Add an example for the
  failure case the agent keeps producing. Re-run the workbench.
- **"We applied poka-yoke and now the tool is annoying to call"**:
  too aggressive. Loosen for low-risk parameters; keep strict on
  high-risk ones (paths, IDs, destructive operations).
- **"Iteration count went up after the redesign"**: the redesign
  added overhead the model can't pay. Re-check format choice (is the
  agent writing escaped JSON now where it was writing markdown
  before?) and length of required fields.
- **"Different agents (Claude, GPT, Gemini) handle the tool
  differently"**: expected. The same tool's description may need
  different framings per model; the principles are universal but the
  examples that calibrate behavior are model-specific.

## Concrete Example

A code-edit tool exists with the signature `edit_file(path,
line_range, content)` requiring exact line numbers from the agent.
Workbench observation: agents miscount line ranges roughly forty
percent of the time, especially after prior edits in the same
session. Iteration count per task is high because the agent keeps
re-reading the file to confirm line numbers. Redesign: replace with
`replace_in_file(path, old_string, new_string)`. Poka-yoke:
`old_string` must be unique in the file (the tool errors if not),
must match exactly (the tool errors if not found), `path` must be
absolute (the tool errors on relative). Description rewritten as a
junior-developer docstring with three examples (single replacement,
multi-line replacement, intentional reject-on-non-unique). Workbench
re-run: iteration count per task drops roughly forty percent, and the
class of "agent edited the wrong lines" failure disappears entirely.
ACI optimization on a single tool produced more value than several
rounds of prompt-tuning on the calling agent — consistent with
Anthropic's reported SWE-bench finding.

## Sources

- `[[concepts/agentic-patterns]]` (section ACI)
- `[[sources/building-effective-agents-anthropic]]`
- Synthesized from Erik S. and Barry Zhang's 2026 Anthropic engineering
  post "Building Effective AI Agents" (Appendix 2, ACI design) via
  Danilo's wiki paraphrase. The replace_in_file example reflects the
  pattern that became standard in Claude Code's file-edit primitives.
