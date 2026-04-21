---
name: harness-garbage-collection-day
description: Use when onboarding team ritual — weekly ritual converting recurring human PR feedback into doc + persistent review agent.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when the team has standing review-agent infrastructure but is still
seeing the same human-comment patterns reappear week after week — the
NFRs-as-prompts machinery exists, but nobody is feeding it. Fires when
a team lead notices that the review queue has the same five comments
this week as last week, or when an engineer says "I keep telling the
agent the same thing in PR comments". The skill installs a weekly
ritual that converts those recurring comments into docs and persistent
review-agent rules so the class of feedback never recurs.

## Preconditions

- Review agents per persona already exist (see
  `harness-review-agents-by-persona`). Without them, GC Day produces
  docs with no enforcement.
- The team can commit a recurring meeting slot — Lopopolo runs his on
  Fridays — where everyone surfaces the slop they hit during the week.
- A repo location exists for the persona docs and review-agent prompts
  (typically `docs/review/` or similar). Without a destination, the
  doc updates have no home.

## Execution Workflow

1. Schedule the slot. A weekly cadence is the floor; teams shipping
   high PR volume sometimes do twice-weekly. Lopopolo: "I essentially
   asked every engineer on the team to take one day a week, Fridays, we
   called it garbage collection day." The day matters less than the
   regularity.
2. Open the slot with a slop list. Each engineer brings two to three
   things they observed during the week — PR comments they had to type
   more than once, agent behaviours they had to correct manually,
   patterns they wished the harness caught.
3. Categorize each item by persona — frontend, reliability, security,
   product, accessibility. If an item does not fit a current persona,
   note whether it deserves a new persona or whether the persona set
   needs adjusting.
4. For each category, decide the surface. Most items become a one- or
   two-line addition to the persona doc plus a tweak to the review
   agent's prompt that points at the new section. Some items are
   structural and become a test (see
   `harness-test-source-code-structure`); some are local and become a
   lint (see `harness-lint-as-prompt`).
5. Write the doc updates and prompt changes during the meeting. The
   ritual is intentionally synchronous — async drafts get deferred to
   "next week" indefinitely. Commit before the slot ends.
6. Before closing, do a five-minute audit of last week's GC items. Did
   the class of feedback drop? If not, the doc edit was too vague or
   the agent prompt change did not actually fire. Fix the surface
   wording before adding new items on top.
7. Track the cumulative slop classes eliminated. Three months in, the
   pattern is visible: the same slot that started with twenty items a
   week is producing five, and the human review queue is unrecognizably
   shorter.

## Rules: Do

- Run the ritual on a fixed cadence. Skipping a week lets the slop
  accumulate to a backlog that resists triage.
- Categorize every item by persona before deciding the surface. The
  persona is what tells you which doc to update and which agent to
  point at it.
- Commit doc updates and prompt changes during the meeting. Delayed
  edits become forgotten edits.
- Audit last week's items every week. The ritual without audit becomes
  a vent session; the audit makes it operational.
- Treat new personas as intentional. If a recurring item does not fit
  any persona, decide whether to add a new persona deliberately, not
  by default.

## Rules: Don't

- Don't run GC Day without review agents already wired. Docs without
  agents to read them are aspirational.
- Don't let the ritual become a complaint session. Every item must
  end in either a doc edit, a prompt change, a new lint, or a new
  test — or an explicit "this is a one-off, no surface needed".
- Don't skip the audit step. Without verification, the ritual loses
  its feedback loop and the harness stops improving.
- Don't accumulate items across weeks. A backlog of slop is a sign
  the slot is too short or the team is not committing to the
  doc-during-meeting rule.

## Expected Behavior

After applying the skill, the human review queue gets visibly shorter
month over month. Comments that used to recur become persona-doc
sections that the review agent enforces; new agents and new humans
inherit the team's accepted patterns through the docs rather than
through tribal lore. The ritual itself becomes a high-leverage hour:
forty-five minutes of slop triage produces enforcement that saves
hours of review time across the following week.

The team's mental model shifts. Recurring review comments stop being
"things to remember to say" and start being "things to encode". The
question reframes from "who taught the agent this?" to "which doc
covers this?".

## Quality Gates

- Slot runs weekly without skip; cancellations are rescheduled, not
  dropped.
- Every item ends with a concrete artifact change (doc, prompt, lint,
  test) or an explicit "no surface needed" rationale.
- Last week's items are audited at the start of this week's slot;
  un-resolved ones are re-triaged before adding new items.
- New personas are added deliberately, with charter and doc.
- A running log of eliminated slop classes is maintained for
  retrospective measurement.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-nfrs-as-prompts` (the parent concept that
GC Day operationalizes), `harness-review-agents-by-persona` (the
machinery the ritual feeds), `harness-lint-as-prompt`, and
`harness-test-source-code-structure` (other surfaces where GC Day
items can land). Methodology phase: 70 (Onboarding-time) — GC Day
is the team ritual that turns lived experience into onboarding
artifacts; also 50 (Qualidade) — the throughput of the quality
machinery is set by the cadence of this loop.

## Output Artifacts

- Updated persona docs in `docs/review/` (or equivalent), per slot.
- Updated review-agent prompts, per slot.
- A running GC log file noting items raised, surfaces chosen, and
  audit verdicts week by week.
- Optional: a quarterly retrospective summarizing the slop classes
  eliminated and the trend in human review-queue length.

## Example Constraint Language

- Use "must" for: weekly cadence, doc edits in-meeting, audit of
  prior items before new triage.
- Use "should" for: persona categorization before surface choice,
  running log of eliminated classes.
- Use "may" for: the exact day of the week, the exact slot length —
  Lopopolo's Friday is a convention, not a rule.

## Troubleshooting

- **"The slot keeps overrunning"**: too many items per engineer.
  Cap at two per person and capture the overflow into next week's
  list, or split the slot by persona on rotating weeks.
- **"Doc edits keep getting deferred to async"**: the in-meeting rule
  is not being enforced. Block the close-of-meeting until the commit
  exists, or pair-edit live with a screenshare.
- **"The same slop keeps showing up week over week"**: the surface
  edit is not firing or the wording is too vague to be acted on.
  Re-read the failure path: where would the agent see this? Adjust
  the surface wording until the test or comment points unambiguously
  at the team's pattern.
- **"Engineers stop bringing items"**: the slot has become a chore.
  Reframe with the audit-first format — show the previous week's
  wins before asking for the new week's items, and the value loop
  becomes visible.

## Concrete Example

It is Friday afternoon. The team gathers for the GC slot. The audit
of last week's items: three of four classes dropped to zero in the
human review queue, the fourth (over-aggressive null checks) is still
recurring because the persona-doc edit was vague — the team rewrites
the section live with a concrete bad-and-good example. New items
this week: three PRs had defensive-coding overuse in the wrong layer
("reliability" persona), two PRs introduced a fresh ORM-style query
helper instead of using the canonical one ("architecture" persona).
The team updates `PRINCIPLES_RELIABILITY.md` with the layered
defensive-coding rule and adds a paragraph to `PRINCIPLES_ARCHITECTURE.md`
naming the canonical query helper, with paths. Both review-agent
prompts get a one-line "see updated section" pointer. Total slot
length: forty minutes. Following week, those classes drop to zero in
the human review queue; the team logs two more eliminated slop
classes in its GC log.

## Sources

- `[[concepts/nfr-as-prompt]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk
  "Harness Engineering" via Danilo's wiki paraphrase. The Garbage
  Collection Day ritual and the human-comment-to-persona-doc loop
  come directly from Lopopolo; the in-meeting commit rule and the
  audit-first format are the harness-pack operationalization.
