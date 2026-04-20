---
name: harness-code-is-free
description: Use when planning backlog, migrations, or refactors in AI-assisted team — shifts scarcity lens from code to attention/context/time.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use whenever a team plans a backlog, a migration, or a refactor and reaches
for the old playbook — "we'll do it gradually over quarters" or "P3s stay
P3s, we'll never get to them" or "let's not refactor that, it's working".
Lopopolo's axiom: "Code is free" — implementation has stopped being the
scarce resource. Fires on phrasing that treats code production as the
limiting factor when the team has agentic capacity available. The skill
re-frames the planning conversation around what is actually scarce now:
human time, attention (human and model), and context window.

## Preconditions

- The team has functioning agentic capacity (Claude Code, Codex, or
  equivalent) and can run multiple agents in parallel.
- Someone owns NFR specification (style, reliability, security) — without
  this, "code is free" produces slop, not progress. See the anti-pattern
  section.
- Backlog or migration scope is concrete enough to fan out — vague
  "improve the codebase" doesn't parallelize.
- The team can review and merge agent output without the human becoming
  the new bottleneck (review agents, hub-and-spoke PRs, etc.).

## Execution Workflow

1. List what the team is currently treating as scarce. Common entries:
   "engineer hours to write the migration", "time to refactor that legacy
   module", "capacity to ship P3s alongside P0s". For each, ask whether
   the scarcity is in implementation (now cheap) or in specification +
   judgment (still scarce).
2. Re-allocate. Items where scarcity was implementation-only get moved
   into the agent fan-out queue. P3s that would never ship under the old
   model get scheduled — fire four agents in parallel, pick the
   implementation that solves the problem best, merge.
3. Take a stuck migration off the shelf. Anything sitting at five percent
   complete for six months because "the last mile is grinding" is exactly
   the case for fanning out — fifteen agents on the remaining endpoints,
   merge winners, close the migration in weeks rather than quarters.
4. Run the large-scale standardization refactor that was always too
   expensive: one ORM, one CI pattern, one concurrency helper, one
   validation library. Pattern variation taxes every future agent's
   attention; standardizing pays back on every subsequent run.
5. Reinvest the freed human time into what is now scarce: writing NFRs
   that make the implicit explicit ("every fetch has retry and timeout",
   "every query that returns >100 rows is paginated"), building review
   agents per persona, drafting persona-oriented docs that describe what
   "good" looks like in this codebase.
6. Localize and internationalize anything that was previously deferred for
   capacity reasons. Day-one i18n in internal tools used to require a
   dedicated sprint — now it ships alongside the feature.
7. Re-audit at the next planning cycle. As the team's NFR coverage
   grows, the fan-out yield improves; as the model improves, more
   categories of work move from "still scarce" to "free".

## Rules: Do

- Treat code production, refactoring, and deletion as cheap. Plan
  accordingly — schedule P3s, close stuck migrations, standardize across
  the codebase.
- Fan out P3s and migration tail-work in parallel and pick the winner.
  Three to five concurrent attempts on a small task usually produces one
  good implementation faster than one careful attempt.
- Move human attention to what is actually scarce: NFR specification,
  review agent design, persona-oriented documentation, system design.
- Standardize aggressively. One ORM, one helper for X, one way to do Y.
  Pattern variation costs tokens on every future agent run.
- Treat localization and accessibility as default, not deferred — capacity
  no longer trades against them.

## Rules: Don't

- Don't read "code is free" as "quality doesn't matter". The opposite is
  true: NFRs become MORE important, because the model picks among the
  hundreds of underspecified decisions in any patch unless you write them
  down. See the anti-pattern section.
- Don't keep P3s in the backlog "for discipline" when fanning them out
  would educate the team about what the product actually needs.
- Don't skip system design because implementation is cheap. Cheap
  implementation exposes design problems that scarce implementation used
  to hide.
- Don't replace human review with no review. Hub-and-spoke PRs and review
  agents preserve judgment; absent review produces fast slop.

## Expected Behavior

After applying the skill, the team's planning conversation moves from
"what can we afford to ship?" to "what should the agents work on next,
and what NFRs do they need to follow?" Stuck migrations get unstuck. P3s
ship in batches. The codebase becomes more uniform — one way to do each
thing — which compounds on every future agent run.

The engineering skill set visibly shifts toward systems thinking,
delegation, and specification. Engineers stop writing the bulk of
implementation code and start writing the rules, examples, and review
agents that make agent output acceptable.

## Quality Gates

- The team has a written list of "what is scarce now" (time, attention,
  context, NFR coverage) — not just "code is free" as a slogan.
- At least one stuck migration or P3 batch has been fanned out and shipped.
- Standardization passes are scheduled or completed (one ORM, one CI
  pattern, etc.).
- NFR documents and review agents exist for the team's top personas
  (frontend, reliability, security, product).
- Localization and accessibility are default-on for new features, not
  deferred.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs with `harness-architecture` (the role separation that makes fan-out safe)
and with `harness-evolution` (each model release expands what counts as
"free"). Methodology phases: 10 (mapping of work), 20 (stack and
standardization decisions), 70 (onboarding the team into the new scarcity
model).

## Output Artifacts

- Updated backlog with fan-out candidates marked.
- Standardization plan (one ORM, one CI pattern, etc.) with target dates.
- NFR documents per persona, committed in the repo.
- Optional: review-agent definitions per persona.

## Example Constraint Language

- Use "must" for: writing NFRs before fanning out work that touches them,
  preserving human review (via agents or hub-and-spoke PRs), naming what
  is scarce in the new lens.
- Use "should" for: parallel fan-out for P3s and migration tail-work,
  large-scale standardization passes, day-one localization.
- Use "may" for: how aggressive the parallelism is (four-way is a sane
  default; some teams run higher), which standardizations to prioritize
  first.

## Troubleshooting

- **"We fanned out and the merged code is inconsistent"**: NFR coverage
  is the missing piece. Each agent picked differently among
  underspecified choices. Add the NFR, re-run, merge.
- **"Reviewers are drowning in agent PRs"**: human review became the new
  bottleneck. Move to hub-and-spoke (no one blocks; the implementation
  agent acknowledges/defers/rejects each comment) and add review agents
  per persona for the categories most often flagged.
- **"We tried to close a stuck migration and it broke production"**: the
  fan-out wasn't gated by the right tests. Test the source code (file
  size, dedup, dependency edges) and the behavior, then fan out.
- **"Code is free, so we should rewrite from scratch every quarter"**:
  no — refactoring and standardizing are free; rewriting throws away
  embedded NFR knowledge. Standardize in place; rewrite only when the
  design itself is wrong.

## Concrete Example

A team has a six-month-stuck migration from REST to GraphQL at five
percent completion. The old playbook ("gradual over quarters, careful
per-endpoint reviews") has not moved the needle since week two. Under
the code-is-free lens: fifteen agents fan out across the remaining
endpoints in parallel, each writing the migration plus the NFR-mandated
retry/timeout/pagination wrapper. Reviewers triage in hub-and-spoke
mode — review agents catch the per-persona NFRs, humans focus on the
two endpoints with subtle business-logic shifts. Migration closes in
two weeks. Separately, twelve P3s that had been frozen in the backlog
since 2025 get fired four-way in parallel; the team merges the best
implementation for each. None of this would have been the right call
when implementation was the scarce resource. The freed engineering
capacity goes into writing NFR docs for two undocumented personas
(reliability and security), which then catch a class of slop in the
next sprint's agent output before it reaches review.

## Sources

- `[[concepts/code-is-free]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- `[[entities/ryan-lopopolo]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk and
  the OpenAI engineering blog post "Harness Engineering" via Danilo's
  wiki paraphrase. The fan-out-P3s and close-the-migration corollaries
  are direct operationalizations of Lopopolo's framing.
