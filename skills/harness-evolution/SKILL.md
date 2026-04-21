---
name: harness-evolution
description: Use when new LLM model releases — re-audit harness, strip obsolete scaffolding, claim new capabilities.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use whenever a major LLM release lands (new Opus, new Sonnet, new GPT,
new Gemini) and a team has a working harness from the prior model
generation. Fires on phrasing like "the harness still works, leave it
alone" or "we'll re-audit eventually" — both are slow paths to a frozen
v1 from six months ago that contains scaffolding the new model no longer
needs and lacks scaffolding it now permits. The skill walks the team
through a structured re-audit and generates a strip-and-claim plan.

## Preconditions

- A harness exists (Planner, Evaluator, context-reset rules, sprint
  contracts, etc.) and was built against an earlier model.
- The new model has been smoke-tested on at least one representative
  task from the team's actual workload — not vendor benchmarks.
- The team can roll back if a strip turns out to have been load-bearing.
  Version-control the harness components themselves, not just the code
  they wrap.
- Someone owns the audit. Without an owner, this never happens.

## Execution Workflow

1. List every component of the current harness in one column: context-reset
   triggers, sprint contracts, Evaluator subagents, lint-as-prompt rules,
   review agents, custom tools, skills. For each, write the assumption it
   encodes about the prior model — "Sonnet 4.5 collapses past 80% context",
   "the model self-rates too leniently on UX", and so on.
2. Smoke-test the new model on a representative task with the harness in
   place. Note where it visibly outperforms or underperforms expectations.
3. For each component, decide one of three: STRIP (assumption no longer
   holds), KEEP (assumption still load-bearing), or EXTEND (component
   could now do more given new capability). Write the decision next to
   the assumption.
4. Strip in small, reversible passes. Remove the explicit context-reset
   trigger first if reasoning depth has grown; remove safety scaffolding
   last. After each strip, run the smoke test again — a strip that
   silently regresses quality is still a regression.
5. Claim new capabilities in equally small passes. If reasoning depth
   permits longer planning horizons, extend the Planner's spec window
   from three sprints to five. If long-context retrieval improved, drop
   resets in favor of compaction inside the same session.
6. Commit the audit as a dated note in the repo: model version, what was
   stripped, what was claimed, what was kept, and the smoke-test results.
   This becomes the changelog the next audit reads first.
7. Schedule the next audit. Tie it to model release cadence, not the
   calendar — auditing on a fixed schedule wastes effort between releases
   and misses the window after a release.

## Rules: Do

- Treat each harness component as encoding a falsifiable assumption about
  the model. Write the assumption down so the audit has something to test.
- Strip in reversible passes. A single big strip is a refactor without
  tests; a sequence of small strips is a controlled experiment.
- Re-run a smoke test after each strip and each claim. Quality regressions
  hide easily under "the harness feels lighter now".
- Date and commit each audit. Future audits start by reading the last one.
- Pair stripping with claiming. The space of useful harness combinations
  doesn't shrink as models improve — it moves.

## Rules: Don't

- Don't preserve scaffolding "just in case" without naming the assumption
  it encodes. Unnamed scaffolding accretes; named scaffolding can be
  audited.
- Don't audit the harness at the same time as a feature push. Conflating
  the two confuses regression sources.
- Don't trust the new model's self-assessment of harness fit. The model
  has no memory of the prior harness's rationale; ask it about specific
  components, not "is this harness still good?"
- Don't claim capabilities that the smoke test didn't actually
  demonstrate. Vendor release notes are marketing input, not evidence.

## Expected Behavior

After applying the skill, the team has a dated audit note listing every
harness component, its prior assumption, and a strip/keep/extend decision
backed by smoke-test evidence. The harness gets lighter where the model
improved and heavier where new capabilities open new ground. The team
stops paying token cost for resets the model no longer needs and starts
exploring planning horizons the prior model could not sustain.

Over time, the audit notes become a model-by-model history that newer
team members can read to understand why the harness looks the way it
does — instead of inheriting v1 scaffolding without rationale.

## Quality Gates

- Every component in the audit has its prior assumption written down.
- Each strip and each claim is tied to a smoke-test result.
- The audit is committed to the repo with the model version in the
  filename or frontmatter.
- The next audit is scheduled (tied to release cadence, not calendar).
- No component remains in the harness without a current rationale.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs with `harness-architecture` (the audit usually revisits the three-role
decision) and with `harness-code-is-free` (which reframes scarcity in a way
that often justifies new claiming moves). Methodology phases: 30
(Skills/Agents) for the architectural changes, 70 (onboarding/team) for
codifying the audit ritual.

## Output Artifacts

- Dated audit note in the repo (e.g., `docs/harness/audit-2026-04-20-opus-4.6.md`)
  listing components, prior assumptions, decisions, and smoke-test results.
- Updated harness components (stripped or extended) committed alongside.
- Optional: a short changelog summary for the team.

## Example Constraint Language

- Use "must" for: dating the audit, writing prior assumptions per
  component, smoke-testing on a representative task before stripping.
- Use "should" for: small reversible passes, pairing strips with claims,
  scheduling the next audit at release cadence.
- Use "may" for: dropping the audit entirely once a model generation
  demonstrably stops needing the prior scaffolding (the audit itself can
  thin out as the harness simplifies).

## Troubleshooting

- **"We stripped a component and quality dropped"**: that component was
  load-bearing. Restore it, mark the strip as "tested, kept", and note
  why in the audit. This is the system working — not a failure.
- **"The new model produces worse output even with no harness changes"**:
  rare but real. Check whether the new model's tool-call format changed,
  whether your prompts encode old model quirks, or whether benchmarks lied
  about the relevant capability. Don't strip in this case.
- **"The audit feels like make-work"**: it is, until the next release
  reveals scaffolding that's been wasting tokens for three months. The
  cost of skipping is invisible until you measure it.
- **"Different team members disagree on what to strip"**: good. Run the
  smoke test with the strip, then without. Disagreement without evidence
  is opinion; disagreement with evidence is data.

## Concrete Example

After Opus 4.6 ships, a team that built its harness on Sonnet 4.5 audits
in one afternoon. The audit note lists fourteen components. The explicit
context-reset trigger that was load-bearing for Sonnet 4.5's context
anxiety is stripped — Opus 4.6's automatic compaction handles the same
window without premature termination. Smoke test confirms no quality
loss across three representative tasks. The Planner is extended: where
Sonnet 4.5 maxed at three-sprint plans, Opus 4.6's reasoning depth makes
five-sprint plans coherent, so the team adds a "longer planning horizon"
option. The Evaluator's few-shot calibration is kept — subjective grading
is still load-bearing. Two review-agent personas are kept; one (the
"check for hallucinated imports" persona) is stripped because Opus 4.6
no longer hallucinates imports at any meaningful rate. Net effect: token
cost per sprint drops by roughly twenty percent; planning quality on
multi-sprint work measurably improves.

## Sources

- `[[concepts/harness-engineering]]` (section "Evolução com o modelo")
- Synthesized from Prithvi Rajasekaran's 2026 Anthropic engineering post,
  paraphrased through Danilo's wiki. The "space of interesting harness
  combinations doesn't shrink — it moves" framing is the central
  takeaway operationalized here as the strip-and-claim audit.
