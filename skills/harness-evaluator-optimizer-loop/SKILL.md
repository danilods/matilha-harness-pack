---
name: harness-evaluator-optimizer-loop
description: Use when task has clear criteria + iterative refinement adds measurable value — generator + evaluator in tight loop.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a task has explicit acceptance criteria and the team observes
that successive rounds of critique produce measurably better output.
Fires when the team finds itself manually re-prompting an LLM with
"this is closer, but the prop types are wrong, also accessibility is
missing" — that manual loop is a hand-rolled evaluator-optimizer that
should be formalized. Distinct from `harness-architecture` (which is
the multi-agent, sprint-contract version of the same idea, used for
long-running work); this skill is the lighter single-LLM version for
tighter loops inside a session.

## Preconditions

- The criteria for "good output" can be articulated explicitly. If the
  team can't say what makes one draft better than another, the
  evaluator has nothing to grade against.
- A human asked to give feedback would articulate the same kind of
  feedback the LLM evaluator can give. If the gap requires expert
  human judgment the LLM cannot reproduce, this pattern won't work.
- Output measurably improves with critique. Test this on a small
  sample before committing to the loop — some tasks plateau after one
  round and the loop adds cost without benefit.
- Loop bounds exist. A max iteration count and a "no new findings"
  termination signal must both be defined.

## Execution Workflow

1. Define the criteria explicitly. Write each criterion as a sentence
   the evaluator prompt can quote — "the component must use semantic
   HTML for interactive elements" beats "good accessibility". Three to
   six criteria is a healthy range; more diffuses the evaluator's
   attention.
2. Build the generator prompt. It produces an initial draft against
   the same criteria the evaluator will grade — exposing the criteria
   to the generator from the start reduces wasted iterations.
3. Build the evaluator prompt. It reads the draft and the criteria,
   produces a per-criterion grade and a list of specific findings
   (not vague encouragement). Calibrate with one or two few-shot
   examples showing the desired feedback shape.
4. Run the loop. Generator produces draft 1; evaluator critiques; if
   any criterion below threshold, generator produces draft 2 against
   the specific findings; repeat. Termination: all criteria pass, OR
   the evaluator finds nothing new versus the prior round, OR max
   iterations reached.
5. Spot-check the evaluator's judgment. Periodically have a human
   compare evaluator findings to their own. When judgments diverge,
   inject a few-shot example to align the evaluator. Drift is the
   main long-term failure mode.
6. Watch for plateau. If draft 3 isn't measurably better than draft 2
   on the criteria, the loop is wasting tokens. Either the criteria
   are too vague (sharpen them) or the task has hit its ceiling
   (terminate earlier).
7. Promote to multi-agent when the loop outgrows a single session —
   when the task is long-running enough that context window becomes
   the limit, switch to `harness-architecture` (multi-agent +
   sprint contract).

## Rules: Do

- Articulate criteria explicitly. The evaluator can only grade what
  the criteria name.
- Expose criteria to the generator from the first draft. Hidden criteria
  produce wasted iterations.
- Calibrate the evaluator with few-shot examples. Without them, the
  evaluator drifts toward generic encouragement.
- Bound the loop. Max iterations and "no new findings" termination
  prevent runaway cost.
- Spot-check evaluator judgment against a human periodically and
  realign with new few-shots when drift appears.

## Rules: Don't

- Don't use this pattern when the task plateaus after one round.
  Single-pass with critique then revision is enough; a formal loop
  adds cost without benefit.
- Don't use the same prompt for generator and evaluator. Same prompt
  produces same blind spots — the evaluator must be tuned for
  skepticism.
- Don't treat evaluator approval as ground truth without spot-checking.
  LLM evaluators drift; a periodic human audit catches drift early.
- Don't run unbounded. Without max iterations, the loop becomes an
  agent loop with worse observability.

## Expected Behavior

After applying the skill, the team has a tight refinement loop where
output measurably improves across iterations against explicit criteria.
The loop terminates cleanly — either all criteria pass, the evaluator
finds nothing new, or max iterations is reached. Iteration count drops
over time as the generator learns (via in-context exposure to the
criteria) to anticipate evaluator findings.

For tasks like component generation, structured writing, configuration
synthesis, and any other work with articulated quality bars, the loop
produces draft three or four output that is visibly better than draft
one — and the team has evidence for the improvement, not just
impression.

## Quality Gates

- Criteria are written explicitly, three to six in count.
- Generator prompt includes the criteria from draft one.
- Evaluator prompt is calibrated with at least one few-shot example.
- Loop has explicit max iterations and "no new findings" termination.
- Periodic human spot-checks compare evaluator judgment to ground
  truth and trigger few-shot updates when divergence appears.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Sits below `harness-architecture` (which is the multi-agent, sprint-contract
evolution of the same idea) and may be wrapped inside
`harness-orchestrator-workers` (each worker can run its own
evaluator-optimizer loop on its slice). Methodology phases: 30 (Skills/Agents
design) and 50 (quality gates), since the loop functions as a self-contained
quality system.

## Output Artifacts

- Criteria spec (3-6 articulated criteria) committed in the repo.
- Generator prompt and evaluator prompt, both committed.
- Evaluator few-shot examples (calibration set).
- Loop output: each draft and its critique, optionally retained for
  audit.

## Example Constraint Language

- Use "must" for: explicit criteria, bounded loop (max iterations and
  termination signal), evaluator calibration with few-shot examples.
- Use "should" for: exposing criteria to the generator from the first
  draft, periodic human spot-checks, retaining drafts for audit.
- Use "may" for: how aggressively to terminate (some teams prefer to
  let the loop run to max iterations even after criteria pass, to
  surface latent issues).

## Troubleshooting

- **"Evaluator approves drafts the team thinks are bad"**: spot-check
  uncovered the divergence. Add a few-shot example showing the bad
  draft and the desired critique. Re-run on a sample to confirm
  realignment.
- **"Drafts plateau after iteration two"**: either criteria are too
  vague or the task has hit its ceiling. Sharpen the criteria first
  (more specific, more measurable). If sharpening doesn't move the
  needle, accept the ceiling and terminate earlier.
- **"Generator keeps making the same mistake despite evaluator
  flagging it"**: the criteria probably name the symptom, not the
  cause. Restate the criterion as the underlying principle, not the
  observable failure.
- **"Loop runs forever"**: missing termination condition. Add max
  iterations and "no new findings" check. Both should be present;
  either alone is insufficient.

## Concrete Example

Code generation task: write a React component for a form with
client-side validation. Criteria: (1) accessibility — semantic HTML,
ARIA where needed, keyboard navigable; (2) state management — no
unnecessary re-renders, state colocated with usage; (3) prop types —
explicit, narrow, exported; (4) error handling — every async path has
a fallback; (5) testability — pure helpers extracted, side effects
isolated. Generator drafts component v1 against the criteria.
Evaluator grades — v1 passes prop types and testability, fails
accessibility (no labels on inputs) and error handling (fetch missing
catch). Generator produces v2 addressing both. Evaluator grades — v2
passes accessibility, partially fails error handling (catch present
but no user-visible feedback). Generator produces v3. Evaluator finds
nothing new — termination. Three iterations, measurably better than
one-shot output, and the team has a record of every critique for
future calibration. Compared to a single-pass generation, the v3
component is production-ready; the v1 component would have required
human revision that the loop instead automated.

## Sources

- `[[concepts/agentic-patterns]]` (section Evaluator-Optimizer)
- `[[sources/building-effective-agents-anthropic]]`
- Synthesized from Erik S. and Barry Zhang's 2026 Anthropic engineering
  post "Building Effective AI Agents" via Danilo's wiki paraphrase.
  The single-session framing here distinguishes this skill from the
  multi-agent sprint-contract architecture in `harness-architecture`.
