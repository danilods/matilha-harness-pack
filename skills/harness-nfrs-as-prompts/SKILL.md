---
name: harness-nfrs-as-prompts
description: Use when team feedback repeats across PRs — convert non-functional requirements into prompt surfaces (lint/test/review-agent messages).
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when reviewers find themselves writing the same comment on PR after PR —
"use parse-don't-validate", "no `unknown` at deep paths", "wrap fetch with
retry", "files over 350 lines need to split". Fires when the human review
queue is dominated by stylistic or structural feedback that an agent could
have applied if it knew the team's choice. The skill reframes those
recurring comments as non-functional requirements (NFRs) that need to live
on prompt surfaces (lints, test failures, review agents) rather than in the
heads of three senior engineers.

## Preconditions

- The team can identify at least three classes of recurring review
  feedback. Without recurrence, this is over-engineering — answer the one
  comment in chat and move on.
- The repo has space for custom lint rules, structural tests, or
  per-persona review agent configs. A repo where nothing custom can be
  added has no place to land NFRs.
- The team accepts that NFRs are the team's choice, not a universal
  truth. Two competent teams will pick different parse-don't-validate
  conventions; both can be right within their codebase.

## Execution Workflow

1. Inventory the recurring feedback. Read the last twenty merged PRs and
   list every comment that appears more than once. Group them by intent
   (defensive coding, type discipline, file structure, error handling,
   etc.). Most teams find six to ten classes of feedback covering 70% of
   their review traffic.
2. For each class, decide which prompt surface fits best. Behaviour-level
   NFRs (await-in-loop, missing retry) usually become lints. Structural
   NFRs (file length, schema dedup) become tests over source code.
   Persona-flavoured NFRs (security review, product spec adherence) become
   review-agent comments. AGENTS.md is the surface of last resort — it
   gets ignored at compaction.
3. Write the surface with the remediation path embedded, not just the
   symptom. A lint that says "await in loop forbidden" is signal without
   guidance; a lint that says "await in loop forbidden — use Promise.all
   for parallel calls; see docs/PERFORMANCE.md#concurrency" is a prompt
   the agent can act on without further investigation.
4. Bias toward just-in-time surfacing over frontloaded coverage. Agents
   read AGENTS.md once and forget; they hit lint and test errors at the
   exact moment the offending code exists. Per Lopopolo, OpenAI: "agents
   have seen every variation in training; you specify the choice or they
   pick arbitrarily" — the surface choice decides whether the
   specification is reachable when it matters.
5. Commit each new surface with a short note in the commit message
   pointing to the recurring feedback class it replaces. Future authors
   need to see the lineage so the surface does not get deleted in a
   cleanup.
6. Re-measure after two to three weeks. The class of feedback that drove
   the surface should drop to near zero in the human review queue. If it
   does not, the surface wording or trigger is wrong — iterate before
   adding the next NFR.
7. Treat surfaces as living artifacts. Reading transcripts and PR threads
   weekly surfaces drift; pair with the garbage-collection ritual (see
   `harness-garbage-collection-day`) to keep the surface set current.

## Rules: Do

- Encode each recurring feedback class on exactly one surface. Multiple
  surfaces saying the same thing in different words confuse the agent
  more than no surface at all.
- Embed the remediation path in the message — what the agent should do,
  not just what it did wrong. Investigation overhead defeats the surface.
- Pick the surface that fires at the right moment. Lints fire locally on
  save; tests fire on push; review agents fire on PR open. Match the
  surface to when the agent can act.
- Attribute each surface to a persona (frontend, reliability, security,
  product) so the agent reads the comment with the right lens.
- Keep AGENTS.md short and stable; push everything specific into surfaces
  the agent will hit at the moment of decision.

## Rules: Don't

- Don't try to encode every possible NFR up front. The set is endless;
  the team's accepted choices are finite. Surface only what reviewers
  actually keep saying.
- Don't reach for AGENTS.md as the default surface. It is high-leverage
  at session start and low-leverage by mid-session compaction. Reserve
  it for cross-cutting principles, not specific patterns.
- Don't write a surface message that requires the agent to read three
  other docs before fixing. The fix should be obvious from the message
  plus one canonical example link.
- Don't mix behavioural and structural NFRs on the same surface. Lints
  for behaviour, tests for structure — see `harness-lint-as-prompt` and
  `harness-test-source-code-structure`.

## Expected Behavior

After applying the skill, the human review queue compresses. Reviewers
stop typing the same comment for the third time and start typing only
the comments that genuinely required human judgment. New engineers and
new agents alike inherit the team's choices through the surfaces rather
than through tribal knowledge.

The shape of the codebase converges. A team that surfaces "single canonical
zod schema per shape" sees zod schemas converge across packages within a
few sprints, without anyone running a manual migration.

## Quality Gates

- Every recurring feedback class from the inventory has exactly one
  surface; no class is duplicated and no class is uncovered.
- Each surface message contains a remediation hint and, where useful, a
  link to a canonical example.
- Surfaces are persona-attributed so the agent reads with the right
  lens.
- AGENTS.md does not duplicate any specific surface. It carries
  cross-cutting principles only.
- The recurring feedback class is re-measured after two to three weeks
  and the surface adjusted if the drop did not happen.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-lint-as-prompt` (the lint flavour of the
same idea), `harness-test-source-code-structure` (the test flavour),
`harness-review-agents-by-persona` (the review-agent flavour), and
`harness-garbage-collection-day` (the ritual that feeds the inventory).
Methodology phase: 50 (Qualidade) — surfaces are how the team's
quality bar reaches the agent automatically; also 70 (Onboarding-time) —
new agents and new humans inherit the bar through the surfaces.

## Output Artifacts

- A short audit doc listing the recurring-feedback inventory and the
  surface chosen for each class, committed to the repo.
- The surface artifacts themselves: custom lint rule files, structural
  test files, review-agent prompts — all version-controlled.
- Optional: a CHANGELOG entry per surface so future authors see the
  lineage.

## Example Constraint Language

- Use "must" for: one surface per recurring class, embedded remediation
  path in the message, persona attribution.
- Use "should" for: re-measuring after two to three weeks, linking to a
  canonical example in the message.
- Use "may" for: which surface (lint vs test vs review agent) when the
  intent could fit more than one — pick the one that fires earliest.

## Troubleshooting

- **"We added the lint and the agent still ignores it"**: the message
  probably names the symptom but not the fix. Rewrite the lint output
  to say what to do, not just what is forbidden.
- **"AGENTS.md is huge but quality has not improved"**: most rules in
  AGENTS.md belong on a more specific surface. Triage each rule against
  the surface table and migrate.
- **"Reviewers complain that the lint is too noisy"**: the surface fires
  in cases the team did not actually intend to forbid. Tighten the
  trigger, add an escape hatch comment, or split into a stricter and a
  warning variant.
- **"We surfaced everything and now CI is unreadable"**: surfaces are
  competing for attention. Categorize them by persona so the agent and
  the human can read the failure log as a structured report rather than
  a wall of text.

## Concrete Example

A team notices the same review comment for three weeks running: "use
parse-don't-validate at the edge". The first instinct is to add it to
AGENTS.md, but AGENTS.md is already long and the comment kept happening
even after a previous addition. The team writes a custom lint that fires
when `unknown` types appear in deep code paths, with the message: "You
have `unknown` here. Parse-don't-validate at the edge — type should come
from the zod schema in `packages/api-types`. Example: `src/handlers/payments.ts:42`."
The lint is wired to fire on save in the editor and on push in CI. Within
two weeks, the class of feedback drops to zero in the human review queue.
A month later a new engineer joins and immediately writes parse-don't-
validate code without anyone telling them — the lint taught them and the
agents they use the team's choice the same way.

## Sources

- `[[concepts/nfr-as-prompt]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- `[[entities/ryan-lopopolo]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk
  "Harness Engineering: How to Build Software When Humans Steer, Agents
  Execute" via Danilo's wiki paraphrase. The 500-micro-decisions framing
  and the surfaces-as-prompts taxonomy come from Lopopolo; the
  inventory-then-surface workflow is the harness-pack operationalization.
