---
name: harness-jit-retrieval
description: Use when agent needs info beyond upfront context — just-in-time retrieval (file paths, queries, tools) vs RAG tradeoffs.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when an agent has to operate over a corpus that does not fit upfront in
its context — a large repo, a docs folder, a knowledge base, a codebase that
churns weekly. Fires when someone proposes "let's chunk and embed everything
and let the agent retrieve top-k", or when the existing RAG pipeline is
returning stale or off-target chunks. The skill walks the team through the
choice between pre-inference RAG, just-in-time agentic search, and the
hybrid pattern that Claude Code itself uses.

## Preconditions

- The agent has access to file-system or query tools (glob, grep, list, read,
  HTTP fetch) — not just an embedding-search tool. JIT retrieval needs real
  navigation primitives.
- The corpus has some structure — meaningful folder names, predictable file
  layout, naming conventions. JIT depends on metadata as signal.
- The team can measure retrieval quality (right doc found, right chunk used)
  on a small set of representative queries. Without measurement, the JIT
  vs RAG debate becomes religious.

## Execution Workflow

1. Characterize the corpus. Static legal text and finance disclosures favor
   pre-inference RAG; a churning codebase or a fast-moving knowledge base
   favors JIT. Mixed corpora favor hybrid. Write the characterization down
   so the choice is auditable.
2. Sketch the agent's typical query shape. If the agent asks for "the policy
   that applies to case X", an embedding search may match well. If the
   agent asks for "the file that defines `processOrder` in this repo",
   targeted glob plus grep beats embeddings every time.
3. If RAG is plausible, prototype with a small chunked index and measure
   retrieval precision and recall on the query set. Note the staleness
   window — how often the index rebuilds versus how often the corpus
   changes. A weekly rebuild on a daily-changing corpus is a 6-day lie.
4. If JIT is plausible, give the agent the right tool set: glob for path
   patterns, grep for content, head and tail for sampling without loading
   whole files, a read tool for surgical full-file reads. Calibrate with a
   few example traces of how the agent should walk the corpus.
5. For most teams, prototype the hybrid pattern: a small upfront file
   (CLAUDE.md, AGENTS.md, or an llms.txt) with stable orientation plus the
   JIT tool set for everything else. Measure against the same query set.
6. Pick the simplest pattern that hits the quality bar. Anthropic guidance
   for context-rich agents is "do the simplest thing that works" — start
   with hybrid, only add embeddings if a measured query class fails.
7. Document the choice and the measurement, so the next agent author or
   the next corpus-change does not relitigate from scratch.

## Rules: Do

- Treat metadata (folder hierarchy, naming conventions, timestamps) as a
  first-class signal. JIT works because the agent reads structure, not
  just content.
- Give the agent tools that let it sample without committing — head, tail,
  list-with-line-counts. Sampling is how the agent avoids burning tokens
  on becos.
- Default to the hybrid pattern for most agents: small upfront orientation
  plus targeted retrieval tools. Pure RAG and pure JIT are corner cases.
- Measure retrieval quality, not just retrieval latency. A fast wrong
  chunk is worse than a slower right one.
- Keep the upfront file small enough to live in the attention budget
  alongside the task — otherwise the JIT savings get spent on orientation.

## Rules: Don't

- Don't deploy pure RAG over a corpus that changes faster than the index
  rebuilds. The agent will silently quote outdated content.
- Don't dump the entire corpus into the prompt as "context insurance".
  This is the most expensive way to lose recall to context rot.
- Don't ship JIT without good navigation tools and example traces. The
  agent will explore unproductively and burn tokens in dead ends.
- Don't pick a retrieval strategy on architecture-debate grounds; pick it
  by measuring on real query shapes from the agent's actual job.

## Expected Behavior

After applying the skill, the team has a named retrieval strategy with a
measured quality baseline, not a vague "we use RAG" or "the agent figures
it out". Token usage on retrieval-heavy tasks becomes more predictable
because the agent walks the corpus the same way each time. Stale-content
incidents drop because dynamic corpora are reached through live tools
rather than a snapshot index.

When a new corpus or a new agent shows up, the team has a default to
reach for (hybrid) and a checklist of when to deviate (static text →
RAG; tightly structured repo → pure JIT). The decision stops being
re-argued every quarter.

## Quality Gates

- Retrieval strategy is named and justified in a doc next to the agent
  definition.
- Measured retrieval precision and recall on a representative query set
  exist before deploy and are re-run after corpus or agent changes.
- The upfront context (if hybrid) fits in roughly two screens — small
  enough that the JIT pattern still pays off.
- The agent's tool set includes navigation primitives, not just a single
  search endpoint. Glob, grep, and sampling tools are the table stakes.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-context-rot-budget` (JIT is the operational
answer to keeping the upfront budget small) and `harness-aci-tool-design`
(JIT only works when the tools are well-shaped). Methodology phase: 30
(Skills/Agents) for the design choice, 40 (Execução) for the daily-driver
pattern as the agent runs.

## Output Artifacts

- A retrieval-strategy doc next to the agent (which pattern, why, what
  was measured).
- Tool definitions for the navigation primitives, committed to the repo.
- Optional: an `llms.txt` or small upfront orientation file when the
  hybrid pattern is chosen.

## Example Constraint Language

- Use "must" for: measuring retrieval quality before deploying a strategy,
  giving the agent navigation primitives when JIT is in play.
- Use "should" for: defaulting to hybrid for mixed or churning corpora,
  documenting the choice so it is not relitigated.
- Use "may" for: layering an embedding search on top of hybrid for a
  specific query class that JIT alone misses.

## Troubleshooting

- **"JIT agent burns too many tokens exploring"**: the navigation tools
  are weak or the corpus structure is opaque. Add naming conventions, an
  index file, or richer metadata; sometimes a single ARCHITECTURE.md
  sketch turns chaotic exploration into directed search.
- **"RAG returns plausible but stale chunks"**: the corpus changes faster
  than the index rebuilds. Either shorten the rebuild cycle to under the
  change cadence, or move the dynamic portion to JIT.
- **"Hybrid is slower than pure RAG"**: it should be — JIT trades latency
  for freshness and precision. Compare quality, not just speed; if RAG's
  faster wrong answers are acceptable, the task probably did not need an
  agent in the first place.
- **"The team wants embeddings because everyone uses them"**: this is the
  fashion-driven case. Run the measurement with hybrid first; if hybrid
  hits the bar, embeddings are an avoidable operational cost.

## Concrete Example

A team is building a research agent over the company's internal docs
folder — about 4000 markdown files, edited by humans, churning weekly.
First proposal is RAG: chunk every file at 500 tokens, embed with a
standard model, index in a vector store, retrieve top 5 per query. The
prototype works, but two issues surface within a week. First, recently
edited docs do not appear until the nightly rebuild. Second, the
embedding model misses semantic edges the team cares about — "incident
playbook" and "runbook" cluster differently than the team uses them.
The team pivots to a JIT approach: the agent uses targeted glob and
grep on `docs/`, then reads the top-matching files in full. The upfront
context is a 60-line `docs/index.md` that names the folder structure
and the canonical doc per topic. Token usage per query rises modestly,
latency rises about a second, but freshness and retrieval precision
both jump well above the RAG baseline. Six months later, when the team
adds a finance-disclosure corpus that is genuinely static and large,
they layer pre-inference RAG just on that corpus while keeping the
docs folder on JIT — a hybrid that matches each corpus to the right
strategy.

## Sources

- `[[concepts/context-engineering]]`
- `[[sources/context-engineering-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Effective context
  engineering for AI agents" (Rajasekaran, Dixon, Ryan, Hadfield) via
  Danilo's wiki paraphrase. The "do the simplest thing that works"
  guidance and the hybrid-as-default framing carry through to harness-pack
  voice with the measurement discipline made explicit.
