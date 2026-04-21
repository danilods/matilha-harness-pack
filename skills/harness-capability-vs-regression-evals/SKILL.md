---
name: harness-capability-vs-regression-evals
description: Use when building eval suite — distinguish capability evals (low pass, target weakness) vs regression evals (~100% pass, detect drift).
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when the team is starting to build an eval suite and is about to
treat it as one undifferentiated bag of tasks, or when an existing
suite is producing scores that confuse "we got better at hard things"
with "we broke something that used to work". Fires when someone asks
"is the new model better?" and nobody can answer without staring at a
spreadsheet for an hour. The skill installs the dual-purpose framing —
capability and regression evals are different beasts with different
target pass rates and different jobs.

## Preconditions

- A grader taxonomy exists or is being designed (see
  `harness-eval-graders-taxonomy`). Without graders, neither capability
  nor regression evals can run.
- The team accepts that the same task can change role over time. A
  capability task that the agent now nails consistently graduates into
  regression — the suite is a moving artifact.
- There is a way to run both suites on demand (CI for regression, on
  model-upgrade for capability). Without an execution surface, the
  framing is theoretical.

## Execution Workflow

1. Define the two suites separately in the eval harness. Capability
   suite = the hill the team is climbing, current pass rate intentionally
   low (commonly 20-50%). Regression suite = the floor the team has
   reached, target pass rate ~100%. Same harness, different sets and
   different purposes.
2. Source capability tasks from current frontier failures — bug
   tracker entries, support escalations, the "we wish the agent could
   do this" backlog. Twenty to fifty tasks per capability area is a
   healthy starting size; more produces noise, less produces lucky
   averages.
3. Source regression tasks from things the agent already does well.
   The first regression suite is often a mix of historical capability
   tasks that have saturated and a handful of "do not break" guardrails
   the team treats as table-stakes.
4. Set the pass-rate expectation explicitly per suite. Capability
   eval at 30% pass is healthy progress; regression eval at 95% pass
   is a bug. The expectation tells the team how to read a score
   without re-deriving the framing every time.
5. Run regression on every change — model upgrade, harness tweak,
   prompt revision, new tool. Run capability on the slower cadence
   that matches the team's improvement cycle (per model release, per
   sprint, per quarter).
6. Manage the graduation path. When a capability eval saturates at
   ~100%, the tasks graduate into the regression suite. The capability
   suite gets refreshed with harder tasks — the hill keeps moving.
7. Treat both suites as living artifacts. A regression suite that
   never changes is missing recently-saturated capability work. A
   capability suite that never refreshes is no longer a hill — it is
   a comfortable plateau.

## Rules: Do

- Maintain capability and regression as separate suites with
  different target pass rates. Mixing them produces an averaged score
  that obscures both signals.
- Source capability tasks from real frontier failures, not from
  imagined hard cases. Imagined difficulty does not match the
  distribution the agent actually fails on.
- Run regression on every change; run capability on the model-release
  or sprint cadence. The cost profile and the signal cadence are
  different.
- Graduate saturated capability tasks into regression. The capability
  suite is the hill; saturated tasks are the floor.
- Refresh the capability suite when it saturates as a whole. A
  capability suite at 100% has stopped pointing at improvement.

## Rules: Don't

- Don't treat the eval suite as one undifferentiated set. The two
  jobs (climb the hill, hold the floor) need different framings to
  read the score correctly.
- Don't expect capability eval to start at high pass rate. A
  starting pass rate of 80% means the suite is not pointing at the
  team's actual frontier.
- Don't ignore a small drop in regression pass rate as noise. A 95%
  regression score means something used to work that no longer does;
  triage rather than averaging.
- Don't refresh the regression suite by replacing tasks. Add
  graduated capability tasks; only remove a regression task when the
  underlying functionality has been intentionally retired.

## Expected Behavior

After applying the skill, the team can answer "is the new model
better?" in two specific senses: the regression suite says whether
anything the agent used to do has broken, and the capability suite
says whether the team's current frontier moved. The two scores
together give a complete picture; neither alone is enough.

Over time, the cycle becomes self-reinforcing. Capability tasks get
nailed and graduate; regression coverage grows; new capability tasks
take their place at the frontier. The suite as a whole tells the
story of what the team has shipped and what it is reaching for.

## Quality Gates

- Capability and regression suites are defined separately with
  explicit, different target pass rates.
- Capability tasks come from real frontier failures with documented
  provenance.
- Regression suite runs on every change; capability suite runs on the
  cadence that matches the team's improvement cycle.
- A graduation policy is documented: when a capability eval saturates,
  it moves to regression and the capability suite refreshes.
- Suite sizes are tracked over time so growth and refresh patterns
  are visible.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-eval-graders-taxonomy` (the grader
choices that power both suites) and `harness-eval-roadmap-0-to-1`
(the nine-step program that sequences capability and regression into
the larger investment). Methodology phase: 50 (Qualidade) — the
two-suite split is what lets the team distinguish improvement from
preservation in their quality story.

## Output Artifacts

- Two separate eval suite definitions in the harness, with target
  pass rates explicitly noted.
- A graduation log capturing which capability tasks moved into
  regression and when.
- A short doc per suite explaining its current frontier (capability)
  or its current floor (regression), refreshed at each graduation.

## Example Constraint Language

- Use "must" for: separate suites with different target pass rates,
  documented graduation policy, regression-runs-on-every-change.
- Use "should" for: capability tasks sourced from real failures, a
  refresh trigger when capability saturates as a whole.
- Use "may" for: the exact saturation threshold (95% vs 100%) — the
  team chooses based on grader noise tolerance.

## Troubleshooting

- **"Capability eval is at 90% on the first run"**: the tasks are
  not at the team's frontier. Source from current bug-tracker
  failures and the "we wish it could do this" backlog rather than
  from generic difficulty.
- **"Regression eval keeps fluctuating between 92% and 98%"**: the
  graders are noisy or the env is not isolated. Triage flakiness
  before treating the score as a signal; a noisy regression suite
  trains the team to ignore it.
- **"We are not sure when to graduate a capability task"**: pick a
  threshold (commonly three consecutive runs at 100%) and document
  it. Without a rule, graduation either never happens or happens
  arbitrarily.
- **"Capability suite never refreshes"**: nobody owns refreshing it.
  Assign the refresh to the same cadence as the model-release loop —
  every release, audit which capability tasks should rotate out.

## Concrete Example

A team launches a code-edit agent and builds an initial capability
eval of thirty tasks drawn from the current bug tracker. The first
model passes 12 of 30. Two model versions later, the agent passes 27
of 30 reliably across three runs. The 27 saturated tasks graduate
into the regression suite, which now stands at 80 tasks total. The
capability suite gets refreshed with thirty harder edge cases pulled
from the most recent month of escalations; the new model passes 8 of
30. Capability and regression now coexist as separate signals: CI
runs the 80-task regression suite on every PR (target ~100%), and
the team runs the 30-task capability suite on each model release
(target: improvement run-over-run). After six months the regression
suite is at 220 tasks and 99% pass rate; the capability suite has
refreshed twice and currently sits at 35%. The team can answer
"better than last release?" in both senses without staring at a
spreadsheet.

## Sources

- `[[concepts/agent-evaluation]]`
- `[[sources/demystifying-evals-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Demystifying
  evals for AI agents" via Danilo's wiki paraphrase. The
  capability-vs-regression dual framing and the graduation path come
  from the original post; the explicit per-suite target pass rate
  and the refresh-on-saturation rule are the harness-pack
  operationalization.
