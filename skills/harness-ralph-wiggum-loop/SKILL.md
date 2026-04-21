---
name: harness-ralph-wiggum-loop
description: Use when agent needs to iterate until multi-reviewer satisfaction — canonical loop structure (implement, self-review, request reviews, respond, repeat).
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when an agent needs to drive a piece of work to multi-reviewer
satisfaction without a human typing "continue" between turns — a feature
implementation that has to clear several specialist review agents, a
refactor that has to satisfy lint, security, and product checks. Fires
when the team notices a human is babysitting the agent through every
review round, or when the loop "works" only because someone is manually
prompting the next step. The skill installs the canonical five-step
self-driven loop and the hooks that keep it moving.

## Preconditions

- Specialist review agents (or scripts that act like them) exist or can
  be created — frontend, reliability, security, product, whatever the
  team cares about. Without reviewers, there is no loop to drive.
- The agent can trigger its own next step via hooks, scripts, or a CI
  pipeline. A loop that depends on a human "continue" defeats the point.
- The team accepts that the loop terminates on reviewer approval, not on
  the agent's own self-evaluation. Self-approval collapses the loop.

## Execution Workflow

1. Define the five canonical steps as code, not as suggestions.
   Implement, self-review locally (lint, type-check, unit tests),
   request review from specialist agents, respond to each piece of
   feedback (accept, defer, or reject with reason), repeat until all
   reviewers approve. Each step is a script or a hook the agent fires.
2. Stand up the specialist review agents. Each has a narrow charter
   (frontend reviews UX and accessibility; reliability reviews error
   paths and observability; security reviews authn/authz and untrusted
   input; product reviews scope and acceptance). Their reports land as
   PR comments or structured files the implementer agent can read.
3. Wire the loop with hooks. Post-implement triggers self-review;
   post-self-review opens or updates the PR; PR open triggers reviewers;
   reviewer comments trigger the implementer agent's response cycle.
   The hooks make the loop self-sustaining.
4. Bias the implementer agent toward "accept the code, not perfection".
   Every review round costs tokens and time; the loop should converge,
   not grow indefinitely. The implementer should accept reasonable
   feedback quickly, defer non-blocking nitpicks with rationale, and
   reject only when a reviewer is wrong about a fact.
5. Watch for "Continue typed by a human is a failure of the harness"
   (Lopopolo's axiom). When the loop stalls and a human types continue,
   that is a hook you forgot to wire — find which transition is
   missing and add it.
6. Cap the iteration count as a safety. A loop that is still going after
   five or six rounds is not converging; surface to a human for a
   decision rather than burning more cycles.
7. After the human merges, review the run. Did each reviewer earn its
   tokens? Did the implementer accept too readily, or push back too
   often? Tune the prompts and the hook chain accordingly.

## Rules: Do

- Make every transition in the five-step loop fire from a hook or
  script, not from a human prompt. The loop owns its own motion.
- Keep specialist reviewers narrow. A frontend reviewer that also
  comments on database indexes is not a specialist — it is a generalist
  with a label.
- Bias toward acceptance of reasonable feedback. The loop's job is
  convergence, not winning every architectural debate at review time.
- Cap rounds at a small number. A non-converging loop is a signal to
  surface, not to push harder.
- Audit the loop after merge. The hook chain and reviewer prompts are
  themselves artifacts that drift; treat them as code that needs
  maintenance.

## Rules: Don't

- Don't let the implementer agent self-approve and skip reviewer
  rounds. The loop's value is exactly the multi-reviewer signal.
- Don't add reviewers without narrow charters. Overlapping reviewers
  produce contradictory comments and the implementer agent thrashes.
- Don't ignore the "human typed continue" failure mode. Lopopolo:
  "Continue typed by a human is a failure of the harness." Treat each
  occurrence as a missing hook to be added, not as an acceptable
  workaround.
- Don't push past the round cap because "we're so close". A loop that
  needs seven rounds usually needed a human decision at round three.

## Expected Behavior

After applying the skill, feature implementations and refactors drive
themselves to merged-PR state with minimal human babysitting. The
human's role compresses to scoping the work upfront and merging at the
end after a 30-second skim. Specialist reviewers catch the things their
charters cover; the implementer agent learns the team's review tone
across rounds.

The team's debugging reframes around the loop itself — when a stall
happens, the question is "which hook is missing" rather than "why did
the agent stop". The harness becomes the artifact under maintenance.

## Quality Gates

- The five-step loop is implemented as hooks or scripts, not as a
  human-driven sequence.
- Each specialist reviewer has a written charter and a return-format
  expectation (where comments land, in what shape).
- An iteration cap exists and triggers a human surface rather than
  silent retry.
- A post-merge audit step records loop length and any "continue typed
  by human" incidents, so the harness improves over time.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-evaluator-optimizer-loop` (the single-LLM
analog when specialist reviewers are overkill) and `harness-architecture`
(the Planner/Generator/Evaluator triangle is the upstream structure that
the Ralph Wiggum loop operates inside). Methodology phase: 40 (Execução)
— this is where work gets driven through reviews to merge.

## Output Artifacts

- Hook or script files implementing the five-step loop transitions,
  committed to the repo.
- Specialist reviewer agent prompts and charters, committed alongside
  the implementer.
- A post-merge audit log noting loop length and stall incidents per
  feature.

## Example Constraint Language

- Use "must" for: hook-driven transitions for every loop step, narrow
  charters per reviewer, an iteration cap that surfaces to a human.
- Use "should" for: bias toward accepting reasonable feedback,
  recording post-merge audit data per loop run.
- Use "may" for: which specific reviewers to include (frontend,
  reliability, security, product, accessibility) — the set varies by
  the team's risk surface.

## Troubleshooting

- **"The loop keeps stalling at the response step"**: the implementer
  agent's prompt does not name how to handle each comment type.
  Explicit instructions for accept-defer-reject with rationale unblock
  most stalls.
- **"Reviewers contradict each other"**: charters overlap. Tighten the
  scopes so each reviewer owns a disjoint slice of the review surface;
  if scopes genuinely conflict (security vs UX), name a tiebreaker
  rule in the implementer's prompt.
- **"Human keeps typing continue"**: Lopopolo: "Continue typed by a
  human is a failure of the harness." Find the transition the human is
  bridging and write a hook for it. Repeat until no human prompt is
  needed in the steady-state run.
- **"The loop ran nine rounds and finally passed"**: the iteration cap
  is missing or set too high. A nine-round loop almost always meant a
  scope or architecture conversation that should have surfaced at
  round three.

## Concrete Example

A feature implementation flows through the loop. The implementer agent
writes about 200 lines of code, commits, and self-runs lint plus unit
tests. A pre-push hook opens the PR; opening the PR triggers four
specialist review agents — frontend, reliability, security, and
product. Each posts structured comments within a few minutes: the
frontend reviewer flags two accessibility issues, the reliability
reviewer flags a missing retry on an external call, security has no
findings, product flags one edge case that the spec did not cover.
The implementer agent reads all comments, acknowledges and fixes the
accessibility and retry findings, marks the product comment as a
follow-up issue with a one-line rationale, and pushes a second commit.
The push retriggers the reviewers. Round two: all four approve. The
human gets a notification, opens the PR, skims the diff and the
review thread for about 30 seconds, and merges. Total elapsed time
from the implementer agent starting to a merged PR: under an hour,
with no "continue" typed by the human.

## Sources

- `[[concepts/agent-centric-codebase]]`
- `[[sources/codex-agent-centric-world-openai]]`
- Synthesized from the 2026 OpenAI engineering post "Leveraging Codex in
  an agent-centric world" (Lopopolo) via Danilo's wiki paraphrase. The
  Ralph Wiggum naming and the five-step loop come from the original
  article; the "Continue typed = harness failure" axiom is Lopopolo's,
  attributed inline.
