---
name: harness-docs-as-system-of-record
description: Use when organizing team knowledge for AI-first workflows — docs/ as system of record with progressive disclosure layout.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team is moving toward AI-assisted development at scale and
realizes that critical knowledge lives in places agents cannot read —
Slack threads, Google Docs, Notion pages, the head of a senior engineer.
Fires when an agent confidently contradicts a team decision because the
decision was never written into the repo, when onboarding a new agent
requires a human to re-explain conventions, or when "we discussed this
already" answers stop being acceptable. The skill installs `docs/` as the
versioned, agent-readable system of record.

## Preconditions

- The team accepts the operational corollary: information that an agent
  cannot reach in context effectively does not exist for the agent. If
  the team is unwilling to migrate knowledge into the repo, this skill
  does not apply.
- A repo with a `docs/` folder (or the freedom to create one) exists,
  and at least one team member can drive the migration.
- An AGENTS.md or CLAUDE.md index exists, or will be created in parallel.
  See `harness-agents-md-as-index` — `docs/` and the index are companions.

## Execution Workflow

1. Inventory the off-repo knowledge. List the Slack channels, Google
   Docs, Notion pages, and tribal-knowledge topics that the team would
   want an agent to know. Categorize them: design decisions, exec plans,
   product specs, reference docs, generated content.
2. Stand up the canonical `docs/` skeleton — `design-docs/` for
   architectural and product decisions, `exec-plans/active/` and
   `exec-plans/completed/` for plan files, `references/` with `llms.txt`
   files for major dependencies, `generated/` for auto-content, plus
   top-level domain docs (DESIGN, FRONTEND, RELIABILITY, SECURITY) as
   the team needs them.
3. Migrate one category at a time. Start with design decisions because
   they are the most-referenced and most-misremembered. Convert each
   off-repo source into a markdown file with a one-paragraph summary, a
   dated decision record, and a link back to the original source for
   provenance.
4. Wire each migrated doc into the AGENTS.md or CLAUDE.md index so the
   agent can reach it. A doc that exists but is unreferenced is almost
   as invisible as a doc that does not exist.
5. Add mechanical guards. A linter checks for orphan docs, missing
   frontmatter, stale ownership. A janitor agent runs nightly or weekly
   to detect rot and open small-fix PRs.
6. Establish the no-regress rule. New decisions go into `docs/` first,
   then get announced in chat — not the other way around. If a decision
   is announced and not written, the next agent run will treat it as
   non-existent. Make this a team norm, not a hope.
7. Re-measure agent behavior on a representative task set after the
   first wave of migration. The success metric is whether the agent now
   cites team-specific decisions correctly without a human in the loop.

## Rules: Do

- Treat `docs/` as the only canonical home for team decisions, plans,
  and references that should affect agent behavior. Other surfaces are
  views or announcements, not the source of truth.
- Date and credit every decision record. "Why we chose X over Y, decided
  by team on date Z" is what makes a decision usable later.
- Keep `docs/exec-plans/` split into `active/` and `completed/` so the
  agent can distinguish current work from history without timestamp
  archaeology.
- Auto-validate. Linters catch orphan pages, missing frontmatter, broken
  links; the janitor opens PRs for stale ownership. Manual upkeep loses
  to scale.
- Cross-reference between `docs/` pages — bidirectional links turn the
  folder into a navigable graph rather than a flat list.

## Rules: Don't

- Don't keep critical decisions in chat. Even a perfect chat search is
  outside the agent's working context; the decision will be lost.
- Don't migrate everything at once. The first wave should be the highest-
  leverage category (usually design decisions); over-eager migrations
  ship as undated, unowned dumps that rot quickly.
- Don't allow `docs/` to become a write-only archive. Without the
  janitor and the linter, structured docs decay just like a monolithic
  AGENTS.md does.
- Don't conflate `docs/` with end-user documentation. End-user docs may
  live in a separate site; `docs/` is the agent's system of record and
  the team's working memory, optimized for both audiences in that order.

## Expected Behavior

After applying the skill, agents stop contradicting team decisions that
were "obviously" decided last quarter. New team members and new agents
onboard from the same surface, and the question "where is this written
down" stops being rhetorical. Decision velocity changes shape — debates
end with a commit to `docs/`, not a Slack message that fades.

The team's notion of "tech debt" expands to include knowledge debt.
Off-repo knowledge becomes a tracked liability that the migration
discharges; the janitor agent prevents fresh accumulation.

## Quality Gates

- The canonical `docs/` skeleton is present and populated for at least
  the first migrated category.
- Every doc has frontmatter naming owner, last-reviewed date, and a
  one-line purpose statement.
- A lint or CI check fails when `docs/` files are orphaned from the
  index or when frontmatter is missing.
- The janitor task runs on a recurring schedule and produces visible
  PRs over time. A janitor that never opens PRs is broken.
- New decisions land in `docs/` before they are announced; the team can
  point to recent examples of this norm being followed.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-agents-md-as-index` (the index points
into `docs/`; the two are designed together) and `harness-jit-retrieval`
(the agent reaches into `docs/` via glob and grep at runtime).
Methodology phase: 70 (Onboarding time) — `docs/` as system of record
is the artifact that makes both human and agent onboarding measurable.

## Output Artifacts

- A populated `docs/` tree with the canonical subfolders.
- Frontmatter conventions documented in `docs/CONVENTIONS.md` or the
  index file.
- Linter and janitor scripts checked into the repo.
- A migration log naming what moved from where, with dates.

## Example Constraint Language

- Use "must" for: dating and crediting every decision record, validating
  orphan pages mechanically, landing new decisions in `docs/` before
  announcement.
- Use "should" for: bidirectional cross-references between docs,
  scheduling the janitor task at least weekly.
- Use "may" for: choosing how aggressively to migrate historical
  knowledge — a partial migration is better than none.

## Troubleshooting

- **"The migration stalled after one wave"**: this is the most common
  failure. Either the team is treating it as a one-time project rather
  than an ongoing norm, or the janitor is missing. Reframe the work as
  ongoing and add the janitor.
- **"Docs are technically there but the agent ignores them"**: the
  docs are unreferenced from the index, or the index itself is bloated
  past its budget. Audit both and fix the navigation surface first.
- **"Owners do not maintain their docs"**: ownership is a fiction
  without a review cadence. Add a frontmatter `last-reviewed` field and
  have the janitor flag any doc whose last review is past a threshold.
- **"Engineers prefer Slack because docs feel heavy"**: the doc template
  is too long. Shrink to a three-section template (decision, why,
  alternatives considered) so the friction matches a Slack thread; the
  permanence is what justifies the move, not extra ceremony.

## Concrete Example

A team adopts AI-assisted development and inventories its off-repo
knowledge. The largest pile: 47 Google Docs of architectural decisions
accumulated over two years. The team migrates them into
`docs/design-docs/<topic>.md` files — each a tight three-section
template (decision, why, alternatives considered) with frontmatter
naming the owner and the original Google Doc URL for provenance. They
add a janitor Codex task that runs nightly: it detects unlinked docs,
missing index entries, and stale frontmatter, then opens two or three
small PRs per week of focused fixes. The AGENTS.md index gets a new
section pointing at `docs/design-docs/index.md`. After four weeks, the
team measures agent navigation accuracy on the team's codebase against
a twenty-task benchmark — accuracy moves from 52% to 87%, mostly
because the agent now reaches the right design decision instead of
inventing one. The team's chat behavior shifts: design debates end
with "PR up to `docs/design-docs/<topic>.md`, please review", not with
a Slack thread that nobody can find next quarter.

## Sources

- `[[concepts/agent-centric-codebase]]`
- `[[sources/codex-agent-centric-world-openai]]`
- Synthesized from the 2026 OpenAI engineering post "Leveraging Codex in
  an agent-centric world" (Lopopolo) via Danilo's wiki paraphrase. The
  invisibility-without-context corollary and the janitor pattern come
  from the original article, carried into harness-pack voice with the
  migration discipline made explicit.
