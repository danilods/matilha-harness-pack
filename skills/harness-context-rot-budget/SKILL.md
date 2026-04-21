---
name: harness-context-rot-budget
description: Use when designing system prompt or agent context — treat tokens as attention budget, not storage; avoid context rot.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team is about to author or rewrite a system prompt, CLAUDE.md, or
agent persona — and the instinct is to "include everything we might need".
Fires when the draft already exceeds a few thousand tokens, when retrieval
quality on the agent has slipped, or when someone asserts "the model has a
200k context window so size doesn't matter". The skill reframes context as a
finite attention budget with diminishing marginal returns and forces the
conversation onto signal density rather than coverage.

## Preconditions

- The team can run a small recall or task-success benchmark on the current
  context (even five hand-graded prompts is enough). Without measurement, the
  budget conversation is hand-waving.
- There is a target task the agent is supposed to do well — not a generic
  "be helpful" mandate. The skill trims toward the task.
- Someone owns the system prompt or context file and can commit edits without
  weeks of review. Context tuning is iterative; ownership friction kills it.

## Execution Workflow

1. Establish the baseline. Read the current system prompt or CLAUDE.md and
   count tokens. Run the agent against five to ten representative tasks and
   record success or recall scores. Save these as the before-state in a file
   the team can reference.
2. Audit each section against signal density. Walk the prompt top to bottom
   and tag each block as guardrail (must stay), canonical example (worth its
   tokens), or coverage padding (low-signal restatement of edge cases). Be
   honest — most prompts have more padding than the author admits.
3. Trim the padding aggressively, in one pass. Replace exhaustive edge-case
   lists with three to five canonical examples that show the shape of the
   right answer. Replace nested if-else policy with one or two clear
   heuristics plus a redirection to the appropriate doc.
4. Re-tune toward the right altitude. Brittle prompts (hardcoded
   if-this-then-that) and vague prompts (high-level platitudes) both fail.
   The trimmed prompt should name the agent's job, the format of acceptable
   output, the small set of guardrails, and stop.
5. Re-measure. Run the same five to ten tasks against the trimmed prompt. In
   most cases recall improves even though token count dropped — that is the
   context-rot signal disappearing.
6. Commit the trimmed prompt with the before/after numbers in the commit
   message. The numbers are the artifact that justifies future trims.
7. Set a recurring audit. Context bloats over time as the team adds rules in
   reaction to incidents. Schedule a quarterly review where each added rule
   either earns its tokens against measurement or gets cut.

## Rules: Do

- Treat the system prompt as an attention budget, not a storage area. Every
  token competes with every other token for the model's focus.
- Keep canonical examples — they teach the shape of the right answer faster
  than rules describing it. A good few-shot earns its tokens many times over.
- Measure recall or task-success against a small benchmark before and after
  any non-trivial trim. Numbers stop the "but what if we need it?" debate.
- Aim for the right altitude: specific enough to guide, flexible enough that
  the model's heuristics still have room to operate.
- Document trim decisions in the commit message so the next author knows why
  the rule they want to add was previously removed.

## Rules: Don't

- Don't equate context window size with context budget. A 200k window is the
  ceiling, not the target. Past a few tens of thousands of tokens, recall
  degrades on most tasks.
- Don't accumulate rules from every past incident in the prompt. Most
  incidents do not generalize, and the rules that codify them rot the most.
- Don't write hardcoded if-else logic into the prompt as a substitute for
  good examples. The prompt becomes brittle and the model loses its
  heuristic grip on the task.
- Don't ship a context trim without re-measuring. Trim-by-feel sometimes
  helps, sometimes hurts; without numbers the team cannot tell which.

## Expected Behavior

After applying the skill, the team has a system prompt that is shorter, more
opinionated, and measurably better at the agent's actual job. The
conversation around future additions changes shape: instead of "should we
add this rule?" it becomes "show me the failed task this rule fixes, and
the recall delta we expect". Most proposed additions die in that question.

Over time, the team builds a small library of canonical examples that earn
their tokens. New agents get composed from those examples plus a thin
guardrail layer, rather than starting from a 4000-token blank-slate prompt.

## Quality Gates

- Token count of the trimmed prompt is recorded alongside the recall or
  task-success delta.
- Each surviving rule has a one-line justification in a comment or sidecar
  file — what task it protects, why a rule beats an example here.
- Canonical examples cover the most common shape of correct output;
  pathological edge cases are not in the prompt.
- The prompt fits on a screen or two when measured at the author's font
  size; if it does not, the audit was not aggressive enough.

## Companion Integration

Complements matilha-ux-pack:cog-cognitive-load at LLM-attention-budget vs human-working-memory
— both skills treat attention as a finite resource and design around its
limits, but cog-cognitive-load addresses human working memory in interfaces
while this skill addresses LLM attention in prompts. Pairs internally with
`harness-aci-tool-design` (tool descriptions also consume budget) and
`harness-jit-retrieval` (the JIT pattern is the operational answer to
"keep the prompt small but reach for more on demand"). Methodology phase:
30 (Skills/Agents) — context shape decisions are made when the agent or
skill is first authored.

## Output Artifacts

- Trimmed system prompt or CLAUDE.md file.
- Before/after recall numbers in the commit message or a small markdown log.
- Optional: a `prompt-audit.md` next to the prompt with per-rule
  justifications for future audit cycles.

## Example Constraint Language

- Use "must" for: measuring before and after a non-trivial trim, removing
  hardcoded if-else policies that mirror existing examples.
- Use "should" for: replacing edge-case lists with canonical examples,
  scheduling a quarterly audit of accumulated rules.
- Use "may" for: deeper rewrites of the prompt structure (XML tags vs
  Markdown headers), keeping a few legacy rules pending the next audit.

## Troubleshooting

- **"We trimmed and recall got worse"**: the trim hit a load-bearing
  example or guardrail. Re-add the most likely candidate and re-measure;
  iterate one section at a time rather than a single big revert.
- **"Every section feels load-bearing"**: this is the most common
  rationalization for keeping bloat. Run the benchmark with a randomly
  removed section — usually nothing moves, which is the proof.
- **"The prompt is fine, the model is just bad"**: try the trimmed prompt
  against a stronger model. If a stronger model also struggles, the prompt
  is under-specified. If a stronger model handles it cleanly, the prompt
  was over-specified for a weaker one.
- **"Engineering keeps adding rules after incidents"**: introduce the
  earn-its-tokens rule. Each new addition needs a failing test case and a
  recall projection; otherwise it goes in a doc the prompt links to, not
  in the prompt itself.

## Concrete Example

A team is designing a system prompt for a code-review agent. The first
draft runs 4000 tokens — every rule the team has ever debated, every
naming convention, every framework opinion. Recall measurement on a ten-
task retrieval benchmark shows 60% accuracy: when asked to apply the
team's "prefer composition over inheritance" rule, the agent often
references an unrelated section about file size limits instead. The team
trims the prompt to 800 tokens — six guardrail rules, three canonical
example reviews showing the agent's expected voice and depth, one
pointer to `docs/code-review.md` for everything else. Recall jumps to
91%. The deleted rules are not lost; they live in the linked doc, where
the agent can fetch them via JIT when relevant. Three months later, an
incident leads someone to propose adding a "no inline SQL" rule back to
the prompt. The audit asks for the failing review, the canonical example
of the rule applied, and a recall projection. The proponent realizes the
existing example covers it and closes the request.

## Sources

- `[[concepts/context-engineering]]`
- `[[sources/context-engineering-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Effective context
  engineering for AI agents" (Rajasekaran, Dixon, Ryan, Hadfield) via
  Danilo's wiki paraphrase. The attention-budget framing and the
  measure-before-trim discipline are the load-bearing parts of the original
  article carried into harness-pack voice.
