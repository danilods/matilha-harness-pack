---
name: harness-eval-graders-taxonomy
description: Use when designing eval for AI agent — code-based vs model-based vs human graders, when each fits.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when the team is sketching its first eval suite for an agent and is
about to default to "we will use an LLM judge for everything", or when
an existing eval is producing scores nobody trusts because the wrong
grader type is doing the wrong job. Fires when someone asks "how do we
measure if the agent is good?" without distinguishing what is
deterministic, what is subjective, and what genuinely needs a human.
The skill installs the three-grader taxonomy and the prioritization
rule that keeps cost and signal in the right ratio.

## Preconditions

- An agent or feature exists (or is being designed) that the team wants
  to measure beyond manual spot-check.
- A small set of representative tasks (commonly 20-50) is available or
  can be assembled from real failures and bug-tracker entries.
- The team accepts that grader design is a first-class engineering
  artifact, not an afterthought. Ad-hoc graders produce ad-hoc scores.

## Execution Workflow

1. List the dimensions you actually need to measure. For a code-edit
   agent: tests pass, no syntax errors, file count expected, code
   style, comment quality, naming. For a research agent: groundedness,
   coverage, source quality, tone. The dimensions name the graders;
   skipping this step produces graders that measure something other
   than what the team cares about.
2. Sort each dimension into the three grader types. Code-based for
   verifiable outcomes (tests pass, file exists, regex matches,
   outcome state in a DB). Model-based for subjective or freeform
   (style, tone, clarity, naming). Human for the gold-standard
   spot-check that calibrates the model-based graders.
3. Apply the priority rule. Code-based where possible, model-based
   where necessary, human judiciously. Most teams that violate this
   end up with a 90% LLM-judge suite that is expensive, slow, and
   lower-signal than a 70% code-based suite would have been.
4. Design code-based graders first. They are cheap and fast — the
   eval suite needs to run on every model release without becoming a
   budget item. Look hard for ways to reframe a "subjective" dimension
   as a verifiable outcome (tests pass instead of "code looks good";
   DB row exists instead of "agent claimed to book").
5. Design model-based graders for the genuinely subjective dimensions.
   Each LLM-judge gets a structured rubric per dimension, an "Unknown"
   output option to prevent hallucination when context is thin, and
   one judge per dimension (isolated) rather than a single judge
   evaluating six things at once.
6. Calibrate the model-based graders against human SME spot-checks.
   Run thirty trials, have an SME grade them, compare. If LLM-judge
   and SME diverge on more than 15% of cases, the rubric is wrong or
   the SME standard is unstated — fix before trusting the score.
7. Reserve human grading for spot-checks (5-10% sample), calibration
   loops, and the truly novel domain where no rubric is yet stable.
   Total cost across a healthy suite is typically 70-80% code, 15-25%
   model, 2-5% human.

## Rules: Do

- Design code-based graders first and only escalate when the
  dimension truly resists verification. Many "subjective" dimensions
  have an objective proxy that nobody bothered to look for.
- Give every model-based grader a structured rubric per dimension and
  an "Unknown" output so it can refuse to score when context is thin.
- Run one LLM-judge per dimension. Multi-dimension judges produce
  averaged judgments that hide the dimension-level signal the eval
  was supposed to surface.
- Calibrate model-based graders against human SME spot-checks before
  trusting the score. Without calibration, the score is a number
  without an anchor.
- Keep total grader cost in proportion. A suite where 90% of cost is
  LLM-judge will be too expensive to run on every model upgrade.

## Rules: Don't

- Don't reach for an LLM-judge by default. The default is code-based;
  LLM-judge is the escalation path, not the starting point.
- Don't grade the path the agent took. Grade the output. Agents
  regularly find valid solutions the team did not anticipate; path-
  graders punish those as wrong.
- Don't skip calibration. An uncalibrated LLM-judge is producing
  numbers that look like measurement and behave like noise.
- Don't use a single LLM-judge across multiple dimensions. The score
  averages and the dimension-level signal disappears.

## Expected Behavior

After applying the skill, the team has an eval suite where each grader
type does the job it is good at. Code-based graders catch the
verifiable failures cheaply and fast on every model run. Model-based
graders surface the subjective regressions that code cannot see, with
calibration that makes the score trustworthy. Human grading exists in
the loop but does not bottleneck it.

The eval suite becomes a thing the team actually runs — on every
model upgrade, on every harness change, on every feature shipped —
because the cost profile is sustainable and the signal is high.

## Quality Gates

- Each measured dimension is mapped to exactly one grader type with
  an explicit rationale.
- Code-based graders cover every dimension that has a verifiable
  outcome.
- Each model-based grader has a structured rubric per dimension and
  an "Unknown" output.
- Model-based graders are calibrated against SME spot-checks; the
  agreement rate is recorded.
- Total grader cost split is roughly 70-80% code, 15-25% model,
  2-5% human.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-capability-vs-regression-evals` (the
dual-purpose framing that decides when graders go into the regression
suite) and `harness-eval-roadmap-0-to-1` (the nine-step program that
sequences grader design into the larger eval investment). Methodology
phase: 50 (Qualidade) — graders are the unit of evaluation that turns
agent behaviour into measurement.

## Output Artifacts

- A grader-design doc listing each measured dimension, its grader
  type, and the rationale for that choice.
- The grader implementations themselves: code-based test files,
  LLM-judge prompts with rubrics, human-grading instructions.
- A calibration log capturing SME-vs-LLM agreement rates per
  model-based grader, refreshed when prompts change.

## Example Constraint Language

- Use "must" for: structured rubric per LLM-judge dimension,
  "Unknown" output option, calibration against SME spot-checks.
- Use "should" for: code-based-first defaulting, one judge per
  dimension, target cost-split ratio.
- Use "may" for: the exact SME-agreement threshold (15%, 10%) — the
  team picks based on the domain's tolerance for grader noise.

## Troubleshooting

- **"The LLM-judge keeps disagreeing with humans"**: the rubric is
  vague or the dimensions are tangled. Split into one judge per
  dimension, sharpen the rubric with two or three concrete examples
  per scoring band, and re-run calibration.
- **"The eval suite is too expensive to run on every model"**: the
  cost split is wrong. Audit which dimensions truly need an LLM-judge
  vs which could be reframed as code-based outcomes. Most suites
  have at least one or two dimensions that escalated unnecessarily.
- **"The code-based grader keeps failing valid solutions"**: the
  grader is checking the path, not the output. Re-write to check the
  end state (DB row, file content, test pass) rather than the
  sequence of tool calls or the exact intermediate text.
- **"Human graders never agree with each other"**: the SME standard
  is unstated. Write a one-page rubric the SMEs share, run a
  calibration round, and revisit until inter-annotator agreement is
  acceptable for the domain.

## Concrete Example

A team builds an eval suite for a code-generation agent. Dimensions:
tests pass (code-based, verifiable), no syntax errors (code-based),
file count matches expected output (code-based), code style
(model-based, hard to verify mechanically), comment quality (model-
based), naming clarity (model-based), spot-check tone (human, 5%
sample). The first version uses a single LLM-judge across style,
comments, and naming and produces low-signal averaged scores. The
team splits into three isolated judges, one per dimension, each with
a four-band rubric and an "Unknown" output. Calibration against an
SME on thirty trials shows 88% agreement on style, 92% on comments,
80% on naming — the naming rubric gets sharpened. Final cost split:
80% code-based, 18% model-based, 2% human. The suite runs on every
model release in under fifteen minutes and produces dimension-level
scores the team trusts.

## Sources

- `[[concepts/agent-evaluation]]`
- `[[sources/demystifying-evals-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Demystifying
  evals for AI agents" (Mikaela Grace, Jeremy Hadfield, Rodrigo
  Olivares, Jiri De Jonghe) via Danilo's wiki paraphrase. The
  three-grader taxonomy and the priority rule come from the original
  post; the cost-split target and the calibration-as-gate framing are
  the harness-pack operationalization.
