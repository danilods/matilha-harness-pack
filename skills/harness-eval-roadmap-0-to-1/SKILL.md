---
name: harness-eval-roadmap-0-to-1
description: Use when starting eval investment from zero — 9-step cascade + Swiss Cheese defense layers (manual testing → bug tracker → reference solutions → balanced sets → harness → graders → transcripts → saturation → maintenance).
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team has shipped an agent with no eval suite and is now
flying blind — every model upgrade is a guess, every regression report
is a he-said-she-said, every capability claim is anecdote. Fires when
someone says "we should really get evals in place" for the third time
without a concrete plan, or when the team has tried to bolt on evals
and produced a brittle suite that nobody trusts. The skill installs the
nine-step cascade for moving from zero to a working eval program plus
the Swiss Cheese defense framing that places eval as one layer among
six.

## Preconditions

- An agent or feature exists that the team wants to evaluate
  systematically. Without a system under test, the roadmap has no
  destination.
- The team can dedicate calendar time across weeks or months — not a
  single sprint. The roadmap is sequential, and skipping steps
  produces the brittle suites the skill exists to prevent.
- Someone owns the eval program. A roadmap with no owner gets stalled
  at step three; a clear owner makes it through the cascade.

## Execution Workflow

1. Step 0 — start early, start small. Twenty to fifty tasks pulled
   from real failures (bug tracker, support escalations, recent
   regressions) is the right starting size. Waiting for a "complete"
   suite is how teams never start.
2. Step 1 — convert manual testing surface into test cases. The bug
   tracker and the support queue already encode what users hit; turn
   those into eval tasks rather than imagining hard cases.
3. Step 2 — write reference solutions and have two SMEs review.
   Tasks where two SMEs cannot agree on pass/fail are too ambiguous
   to use. A 0% pass@100 score after this step usually means the task
   is broken, not the agent incapable — double-check before blaming
   the model.
4. Step 3 — balance the problem set. Cover both "agent should do X"
   and "agent should not do Y". One-sided sets produce one-sided
   optimization. A web-search eval needs both "should search" and
   "should not search" cases.
5. Step 4 — build the harness with isolated env per trial. Shared
   state across trials creates correlated failures that look like
   model issues; the canonical example is an agent gaining unfair
   advantage by reading git history of prior trials.
6. Step 5 — design graders thoughtfully (see
   `harness-eval-graders-taxonomy`). Code-based where possible,
   model-based where necessary, human judiciously. Grade the output
   not the path; allow partial credit for multi-component tasks; LLM
   judges get an "Unknown" output and a structured rubric.
7. Step 6 — read the transcripts. This is the step teams skip and
   regret. A first transcript pass typically surfaces broken graders,
   valid agent solutions penalized as wrong, and grader-bypass
   exploits. Reading is how you know your eval measures what it
   claims to measure.
8. Step 7 — monitor saturation. A capability suite at 100% pass has
   stopped pointing at improvement; graduate the saturated tasks to
   regression (see `harness-capability-vs-regression-evals`) and
   refresh the capability suite with harder cases.
9. Step 8 — maintain the suite. Eval suites are living artifacts;
   ownership, contribution rules, and a refresh cadence keep them
   useful. Pair with the broader Swiss Cheese model — eval is one of
   six layers (automated evals, production monitoring, A/B testing,
   user feedback, manual transcript review, systematic human
   studies). Effective teams combine layers because no single layer
   catches everything.

## Rules: Do

- Start small with twenty to fifty tasks drawn from real failures.
  Imagined hard cases produce a suite that does not match the
  distribution the agent fails on.
- Write reference solutions and run two-SME review before adding a
  task. Ambiguous tasks become noise in every future score.
- Isolate env per trial. Shared state contaminates the score and
  hides real failures behind correlated ones.
- Read the transcripts on every meaningful change. Grader bugs are
  invisible to score-only review; reading is the only defense.
- Graduate saturated capability tasks into regression and refresh
  the capability suite with harder cases.

## Rules: Don't

- Don't wait for a "complete" suite before starting. Reverse-
  engineering eval criteria from a live system is much worse than
  starting with twenty failures.
- Don't grade the agent's path. Grade the output. Path-graders
  punish valid solutions the team did not anticipate.
- Don't trust an LLM-judge score without calibration against an SME
  spot-check. An uncalibrated judge produces numbers without an
  anchor.
- Don't treat eval as the only quality layer. The Swiss Cheese
  framing is the right one — combine eval with monitoring, A/B,
  user feedback, transcript review, and human studies.

## Expected Behavior

After applying the skill, the team has a working eval program within
weeks rather than quarters. Each step in the cascade lands a useful
artifact (a starter task set, a graded harness, a reading habit, a
graduation policy) so the program produces value before it is
"complete". The team can answer "did this change help" with a
specific number and the transcripts to back it up.

Over time, eval becomes part of the development culture rather than
a thing-to-do-eventually. New features come with eval tasks; model
upgrades come with eval runs; quality conversations move from "feels
better" to "regression at 99.2%, capability at 38%, here are the
five transcripts I read".

## Quality Gates

- Twenty to fifty tasks exist within the first calendar month of
  starting the program.
- Every task has a reference solution and two-SME pass/fail
  agreement.
- The problem set is balanced (positive + negative cases) by step 3.
- Trials run in isolated env; shared-state failures are recognized
  and fixed.
- Transcripts are read on every meaningful change (new model, new
  prompt, new tool); grader bugs caught this way are logged.
- A graduation policy exists; saturated capability tasks move to
  regression on a documented trigger.
- The Swiss Cheese framing is documented; the team can name which
  defense layer catches which class of failure.

## Companion Integration

Complements matilha-ux-pack:ux-swiss-cheese-errors at agent-eval-defense vs human-error-tolerance.
The ux-pack version applies James Reason's Swiss Cheese model to UI
error tolerance for human users — overlapping defenses that catch a
user's slip before it becomes a real-world failure. This version
applies the same model to agent eval defense: automated evals,
production monitoring, A/B testing, user feedback, manual transcript
review, and systematic human studies are the six layers, and the
team's job is to combine them so an agent failure that slips one
layer is caught by another. Same model, different application
domain. Methodology phase: 50 (Qualidade) — this skill sits at the
heart of the team's quality investment, sequencing how an eval
program comes into existence.

## Output Artifacts

- A starter eval task set (20-50 tasks) with reference solutions and
  SME-agreement notes, committed to the repo.
- An eval harness with isolated-env-per-trial wiring.
- Grader implementations covering each measured dimension.
- A transcript-reading log capturing what was read, what was found,
  and what was fixed per session.
- A graduation policy doc and a Swiss Cheese defense layer map.

## Example Constraint Language

- Use "must" for: real-failure-sourced starter set, two-SME reference
  agreement, isolated env per trial, transcript reading on every
  meaningful change.
- Use "should" for: balanced problem set by step 3, documented
  graduation trigger, Swiss Cheese map.
- Use "may" for: the exact starter size (20 vs 50), the calibration
  cadence, the graduation threshold — the team picks based on
  domain noise tolerance.

## Troubleshooting

- **"We have 200 tasks but the score is meaningless"**: most likely
  step 2 was skipped — tasks lack reference solutions or two-SME
  agreement. Triage: tasks where SMEs disagree get rewritten or
  removed; the suite shrinks but the score becomes trustworthy.
- **"The eval keeps showing high pass rate but users still complain"**:
  the suite is missing the failure modes users actually hit. Pull
  twenty more tasks from the last month of support tickets and
  re-run; expect a sharp pass-rate drop and a clearer picture.
- **"Transcript reading takes forever and finds nothing useful"**:
  the team is reading random samples. Read the failures first, then
  the surprising successes; both classes surface grader bugs and
  unanticipated valid paths faster than random sampling.
- **"We saturated the capability suite and lost momentum"**: the
  graduation step is missing. Move saturated tasks into regression,
  refresh the capability suite from the latest frontier failures,
  and the program restarts climbing.

## Concrete Example

A team starts an eval program from zero. Step 1: pull thirty tasks
from the last six weeks of bug-tracker entries. Step 2: write
reference solutions and run two-SME review; six tasks are too
ambiguous and get rewritten, two get removed. Step 3: balance the
set with eight "should not do" cases. Step 4: build the harness
with one container per trial — clean filesystem, fresh DB, no
shared cache. Step 5: code-based graders for verifiable outcomes,
plus two model-based graders with structured rubrics. Step 6: read
twenty transcripts the first week — find four broken graders, two
valid solutions penalized, one grader-bypass exploit. Fix all six.
Step 7: monitor pass rate; suite saturates partially and graduates
eight tasks into regression. Step 8: assign an owner, document
contribution rules, refresh capability quarterly. Six months in: 240
tasks across capability and regression, transcript-reading is a weekly
habit, and the team has named which Swiss Cheese layer catches which
class of failure. Eval-driven development is now culture.

## Sources

- `[[concepts/agent-evaluation]]`
- `[[sources/demystifying-evals-anthropic]]`
- Synthesized from the 2026 Anthropic engineering post "Demystifying
  evals for AI agents" (Mikaela Grace, Jeremy Hadfield, Rodrigo
  Olivares, Jiri De Jonghe) via Danilo's wiki paraphrase. The
  nine-step roadmap and the Swiss Cheese six-layer model come from
  the original post; the per-step quality-gate framing and the
  cross-pack pairing with `ux-swiss-cheese-errors` are the
  harness-pack operationalization.
