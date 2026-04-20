---
name: harness-agents-md-as-index
description: Use when writing AGENTS.md or CLAUDE.md — keep as ~100-line index, move authoritative content to docs/ structure.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team is creating or rewriting an AGENTS.md, CLAUDE.md, or any
agent-orienting file at the root of a repo. Fires when the file is already
past a few hundred lines and growing, when the team admits "the agent
ignores most of it anyway", or when someone proposes to put every team
convention into a single big agent-context file. The skill installs the
index pattern: a short file at the root that is pure navigation, pointing
to a structured `docs/` tree where the authoritative content lives.

## Preconditions

- The team accepts that AGENTS.md or CLAUDE.md is read by the agent on
  every turn or session start. Without that acceptance, the size argument
  goes nowhere.
- A `docs/` folder exists or can be created. The index pattern is
  meaningless if there is no destination for the moved content.
- Someone has authority to delete content from the root file. Refactor
  attempts that only "consolidate" tend to grow the file rather than
  shrink it.

## Execution Workflow

1. Measure the current state. Count lines and tokens of the existing
   AGENTS.md or CLAUDE.md. Note the agent's behavior on a small set of
   tasks — does it cite the file, ignore it, contradict it. The before-
   state is the baseline the refactor improves on.
2. Stand up the destination. Create or audit `docs/` with the canonical
   subfolders the team will actually use: `design-docs/` for decisions,
   `exec-plans/` for ephemeral or completed plans, `product-specs/` for
   product specs, `references/` for `llms.txt` of dependencies,
   `generated/` for auto-content, plus top-level domain docs (DESIGN,
   FRONTEND, RELIABILITY, SECURITY) as the team needs them.
3. Walk the existing root file and tag each section. "Pure navigation"
   stays. "Stable principle worth keeping at the root" stays. Anything
   else moves to the appropriate `docs/` location, with a one-line
   pointer in the root file replacing it.
4. Rewrite the root file as an index — about 100 lines, structured as
   pointers. Each pointer names the target file and the kind of question
   it answers. The agent should be able to skim the index and reach the
   right doc in one hop.
5. Cross-link aggressively. Each `docs/` page should link back to the
   index entry that points to it; broken links should fail CI. The
   bidirectional links are the agent's navigation surface.
6. Add mechanical validation. A linter or janitor task checks that every
   `docs/` file is referenced from the index, that no orphan pages
   accumulate, and that the index stays under the line budget.
7. Re-run the original tasks against the new structure. Track whether
   the agent now reaches the right doc on the first hop. The metric the
   refactor is optimizing is "does the agent find the right info", not
   "is the file shorter".

## Rules: Do

- Keep the root file at roughly 100 lines and treat that as a hard ceiling.
  The number is a forcing function — it makes the index actually be an
  index.
- Make every entry in the index a pointer to a specific file with a
  one-line description of what the file answers. No prose paragraphs
  in the index.
- Move authoritative content to `docs/` and let the index be deletable
  per section without information loss. If a section cannot be deleted,
  it is content, not navigation.
- Validate mechanically: linter for orphan pages, link-checker, line-
  count guard on the index. Without enforcement, the index drifts back
  into a monolith.
- Treat the index as a living artifact — it should change every time the
  `docs/` structure does, and the index change should be in the same PR.

## Rules: Don't

- Don't let the index host content "just for now" — once content lands
  in the index it never leaves on its own.
- Don't keep the monolith and add an index next to it. The two compete
  for the agent's attention; one of them will win and it will not be
  the one you wanted.
- Don't mix tonal sections (values, philosophy) with operational sections
  (file paths, conventions) at the root. Move the tonal content to a
  named doc and let the index point to it.
- Don't validate the structure with humans only. Mechanical checks catch
  the drift that humans rationalize away.

## Expected Behavior

After applying the skill, the agent reaches the right doc on the first
hop most of the time, and the team's debate about "what should we add
to AGENTS.md" reframes into "where in `docs/` does this belong, and
does the index need a new pointer". The root file stops being a place
to dump every team opinion.

Onboarding a new agent or a new model becomes faster — the index gives
the agent a one-screen orientation, and the structured `docs/` lets it
fetch only what the current task needs. Token usage on agent startup
drops measurably; recall on cross-cutting questions improves.

## Quality Gates

- Root AGENTS.md or CLAUDE.md is at or under approximately 100 lines.
- Every entry in the root file points to a specific file in `docs/` (or
  another canonical location) with a one-line description.
- Every page under `docs/` is reachable from the index in at most two
  hops. Orphan pages fail the lint.
- A periodic janitor check exists (manual or automated) that re-verifies
  the index against the actual `docs/` tree.
- The agent's first-hop accuracy on a representative task set is recorded
  before and after the refactor.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-docs-as-system-of-record` (the destination
the index points to) and `harness-jit-retrieval` (the index plus
glob/grep is the canonical hybrid retrieval pattern). Methodology phase:
20 (Stack) and 30 (Skills/Agents) — the index pattern is part of repo
scaffolding and of the agent's context strategy.

## Output Artifacts

- A rewritten AGENTS.md or CLAUDE.md at the root, under the line budget.
- A populated `docs/` tree with the moved content, organized into the
  canonical subfolders.
- A linter or janitor script that enforces the index invariants.
- A short before/after metric noting agent first-hop accuracy or token
  usage on a representative task.

## Example Constraint Language

- Use "must" for: keeping the root file under the line budget, making
  every entry a pointer, mechanically validating orphan pages.
- Use "should" for: bidirectional links between index and docs,
  recording before/after agent metrics on the refactor.
- Use "may" for: choosing which canonical subfolders to instantiate
  first — the full set is aspirational and a small repo can start with
  two or three.

## Troubleshooting

- **"The team keeps adding sections back to the root"**: the line
  budget is not enforced. Add a CI check that fails the build when the
  root file exceeds the threshold. The friction is the point.
- **"The agent still ignores the index"**: the entries are too vague or
  the file is still too long. Tighten each pointer to one sentence
  naming the question it answers, and re-trim if the file slipped past
  the budget.
- **"Docs in `docs/` get stale"**: add a janitor task that detects
  unreferenced or untouched docs and opens small-fix PRs. Without the
  janitor, structured docs rot just as fast as monoliths.
- **"We have legitimate cross-cutting principles that belong at the
  root"**: the index can host a tiny "principles" pointer to a single
  `docs/PRINCIPLES.md`. The principles file lives in `docs/`, not at
  the root.

## Concrete Example

A team's AGENTS.md is 800 lines and growing — accumulated over a year
of "the agent should also know X" additions. It covers how the team
builds, the team's values, every non-functional requirement, all
naming conventions, three competing approaches to error handling.
Symptom: about 30% of agent runs ignore the file entirely (context
anxiety from the size), and another 30% cite outdated sections that
were never updated when the team's practices changed. The team
refactors. The new AGENTS.md is 95 lines: a one-paragraph orientation,
then four sections of pointers — `docs/design-docs/index.md` for
decisions, `docs/exec-plans/active/` for current work, `docs/RELIABILITY.md`
for reliability NFRs, `docs/CONVENTIONS.md` for naming and style. The
800 lines of moved content land in those files, deduplicated and
dated. A linter rejects PRs that push the root file past 100 lines or
add an orphan doc. After two weeks, agent first-hop accuracy on a
twenty-task benchmark moves from 41% to 78%, and token usage on
session start drops by roughly 70%. The team's debate about adding new
rules now starts with "which doc does this belong in" rather than "do
we add it to AGENTS.md".

## Sources

- `[[concepts/agent-centric-codebase]]`
- `[[sources/codex-agent-centric-world-openai]]`
- Synthesized from the 2026 OpenAI engineering post "Leveraging Codex in
  an agent-centric world" (Lopopolo) via Danilo's wiki paraphrase. The
  index-vs-monolith framing and the four predictable failure modes of
  the monolith come from the original article carried into harness-pack
  voice with the line-budget enforcement made mechanical.
