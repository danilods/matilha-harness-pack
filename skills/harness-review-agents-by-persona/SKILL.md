---
name: harness-review-agents-by-persona
description: Use when scaling PR review — frontend / reliability / security / product agents per persona, each with its own doc + tuned skepticism.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when human review has become the bottleneck on PR throughput — the
team is shipping enough patches per day that one or two senior reviewers
cannot keep up, or the same reviewer is asked to judge frontend craft,
reliability, security, and product fit on every PR. Fires when a PR sits
open for hours waiting on a single human, or when reviewer fatigue means
genuine issues slip through alongside style nits. The skill installs
specialist review agents per persona, each with a narrow charter and a
doc that anchors its judgment.

## Preconditions

- The team can articulate at least three distinct review personas it
  cares about (commonly frontend, reliability, security, product). A
  team that can only name "general code quality" is not ready — the
  charters have to be distinct.
- There is a way to wire agents into the PR pipeline (PR-open hook,
  comment bot, CI job). Without a wiring point, review agents become
  scripts a human has to remember to run.
- The team accepts that review agents post and do not block. Hub-and-
  spoke means each comment requires a deliberate response from the
  implementer — acknowledge, defer, or reject — not an automatic merge
  veto.

## Execution Workflow

1. Name the personas. Three to five is the right range — fewer means
   one persona will sprawl and become a generalist; more produces
   contradictory comments. Common starter set: frontend architect,
   reliability, security, product. Add accessibility or performance
   when the team's risk surface justifies it.
2. Write a persona doc per agent. Each doc contains the lens (what
   the persona looks at), the team's accepted patterns (parse-don't-
   validate, retry+timeout on every fetch, etc.), and at least three
   canonical examples of good and bad. The doc is what the agent reads
   to ground its judgment; without it, the agent reverts to generic
   best practices that may not match the team.
3. Calibrate skepticism per persona. A security reviewer should be
   high-skepticism on untrusted input and authn paths, low on
   stylistic choices. A product reviewer should be high-skepticism on
   spec adherence, low on file structure. Wrong calibration produces
   either rubber-stamp or reflexive blocking.
4. Wire the agents to fire on PR open and on each subsequent push.
   Comments land as PR comments or structured files the implementer
   agent can read. Ensure none of the agents have merge-block authority
   by default — they post, the implementer responds.
5. Define the implementer agent's response protocol. For each
   comment: accept (push a fix), defer (with one-line rationale and a
   follow-up issue), or reject (with one-line rationale citing fact or
   spec). Per Lopopolo, OpenAI: "every continue typed by a human is a
   failure of the harness" — the response protocol exists so the loop
   does not stall waiting for human arbitration on routine comments.
6. Watch the first ten PRs through the new pipeline. Note which
   agent's comments get accepted, deferred, or rejected most often.
   High accept rate means the agent is well-tuned; high reject rate
   means the persona doc is wrong about the team's choice.
7. Update each persona doc weekly during the garbage-collection
   ritual (see `harness-garbage-collection-day`). The docs are the
   source of truth that the agents read; drift in the docs becomes
   drift in the review judgment.

## Rules: Do

- Keep personas narrow. A frontend agent that comments on database
  indexes is a generalist with a label, not a specialist.
- Anchor each agent in a persona doc with examples. Without a doc,
  the agent invents its own definition of good.
- Wire agents to post-not-block by default. Hub-and-spoke means the
  implementer agent has authority to defer or reject; reviewers are
  advisors, not gatekeepers.
- Calibrate skepticism per persona. Security high on auth, low on
  style; product high on spec, low on file shape. Wrong calibration
  defeats the specialization.
- Update persona docs as the team's patterns evolve. The doc is the
  prompt; stale docs produce stale reviews.

## Rules: Don't

- Don't add a new persona without a charter and a doc. Charterless
  reviewers overlap and contradict each other; the implementer agent
  thrashes.
- Don't give review agents merge-block authority by default. The
  hub-and-spoke value disappears when one reviewer can stall the loop
  unilaterally.
- Don't let the implementer agent silently ignore comments. Every
  comment needs an explicit accept/defer/reject so the audit trail
  is real.
- Don't rely on a single super-reviewer agent. The reason this skill
  exists is that one generalist reviewer cannot catch what four
  specialists can; recombining them undoes the design.

## Expected Behavior

After applying the skill, PR cycle time compresses dramatically.
Specialist comments arrive within minutes of PR open; the implementer
agent processes the comment list in a single pass, fixing or
rationalizing each, and re-pushes. Human reviewers spend their time
on architectural and scoping judgments rather than on style and
defensive-coding nits.

Over time, each persona doc becomes a living artifact of the team's
chosen patterns. A new engineer reading the docs gets a faster
onboarding than a new engineer reading the codebase.

## Quality Gates

- Each persona has a written charter and a doc with canonical examples.
- Skepticism is tuned per persona — security high on auth, low on
  style; product high on spec, low on file shape.
- Review agents post and do not block by default; the implementer
  agent has the response authority.
- Implementer responses are explicit (accept / defer with reason /
  reject with reason) for every comment.
- Persona docs are updated at least weekly during the GC ritual.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-nfrs-as-prompts` (review agent is one
surface among many), `harness-garbage-collection-day` (the ritual that
keeps persona docs fresh), and `harness-ralph-wiggum-loop` (the
five-step loop that drives implementer-reviewer-implementer cycles).
Methodology phase: 50 (Qualidade) — review agents are first-line
quality enforcement; also 70 (Onboarding-time) — persona docs become
the team's accepted-pattern library for new engineers and new agents.

## Output Artifacts

- One persona doc per review agent (frontend, reliability, security,
  product, etc.) committed to the repo.
- Review agent prompts referencing each doc, also version-controlled.
- A short PR-pipeline config showing which agents fire on what trigger.
- Optional: a per-PR audit log capturing comment counts and
  accept/defer/reject rates for harness tuning.

## Example Constraint Language

- Use "must" for: persona doc per agent, post-not-block default,
  explicit implementer response per comment.
- Use "should" for: skepticism calibration per persona, weekly doc
  refresh.
- Use "may" for: which exact persona set to start with — the team's
  risk surface decides whether accessibility, performance, or
  documentation deserves its own agent.

## Troubleshooting

- **"Agents post contradictory comments"**: charters overlap. Tighten
  scopes so each persona owns a disjoint slice of the review surface;
  if the conflict is genuine (security vs UX), name a tiebreaker rule
  in the implementer's prompt.
- **"Implementer agent accepts every comment without thinking"**: the
  response protocol is under-specified. Require a one-line rationale
  for accept too, not just for defer or reject — the rationale forces
  judgment.
- **"Persona doc is huge and the agent ignores most of it"**: the doc
  has crept past the agent's effective attention budget. Split into
  the canonical patterns (must-read) and an examples appendix
  (read-on-demand).
- **"Reviewers find nothing on most PRs"**: skepticism may be too low,
  or the patterns are already so well-encoded in lints that PR review
  is the wrong surface. Re-check whether the persona belongs at lint
  time instead.

## Concrete Example

A PR opens against the team's main repo. Within ninety seconds, four
review agents post in parallel: frontend-architect flags two
accessibility issues (missing aria-label, contrast ratio under 4.5)
and one design-token mismatch; reliability flags a missing retry on
an external fetch and a swallowed exception in the error handler;
security scans the diff for PII handling and finds nothing; product
notes that the spec called for graceful degradation when the upstream
service is offline, which the implementation does not handle. The
implementer agent reads the eighteen total comments in a single pass:
acknowledges and fixes the twelve clearly-actionable items in two
minutes of work, defers the four design-token cases ("blocked on
token refresh, follow-up issue #1247"), rejects two duplicates with
a one-line rationale. The push retriggers the agents; round two, all
four approve. Total elapsed time from PR open to merge-ready: under
ten minutes, with the human's role compressed to a thirty-second skim
before merge.

## Sources

- `[[concepts/nfr-as-prompt]]`
- `[[concepts/harness-engineering]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk
  "Harness Engineering" via Danilo's wiki paraphrase. Per-persona
  reviewers and the hub-and-spoke PR model come from Lopopolo; the
  implementer-response protocol (accept/defer/reject) is the
  harness-pack operationalization.
