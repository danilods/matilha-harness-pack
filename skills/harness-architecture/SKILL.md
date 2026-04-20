---
name: harness-architecture
description: Use when designing multi-agent system for long-running tasks — Planner/Generator/Evaluator + sprint contract architecture.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a team is about to spin up an agent (or set of agents) to build
something that will not fit inside a single short session — an MVP, a
multi-feature backend, a redesign that takes many sprints. Fires when someone
sketches "we'll just point Claude Code at the repo and let it run" without
naming who plans, who builds, and who judges. The skill installs the
GAN-inspired three-role architecture (Planner, Generator, Evaluator) and a
sprint contract before any code is written.

## Preconditions

- The work is genuinely long-running. For one-shot edits or small features,
  this architecture is overkill — say so and stop.
- The team can stand up at least two separate agent processes (subagent,
  sibling session, or worktree). A single LLM impersonating both Generator
  and Evaluator defeats the design.
- There is a way for the Evaluator to interact with the live system, not
  just read code (Playwright MCP, HTTP client, fixture replay, etc.).
- The team can articulate at least three to five subjective quality criteria
  they care about, even loosely — these become Evaluator criteria later.

## Execution Workflow

1. Name the three roles and where each runs. Planner is usually a single
   high-context call (or a chat session) that turns one to four sentences
   from the human into a full product spec. Generator is the coding agent
   in a session or worktree. Evaluator is a separate subagent or sibling
   session — never the same process as Generator.
2. Have the Planner draft the spec. Push it to be ambitious in scope and
   high-level in technical choices — fine-grained tech decisions made too
   early cascade into implementation errors. Save the spec as a file in the
   repo (not chat history).
3. Before any feature code, run the sprint-contract negotiation. Generator
   and Evaluator exchange drafts of "what does done look like for this
   sprint?" until both sides agree. The contract names testable criteria,
   binary thresholds per criterion, and the surfaces the Evaluator will
   touch (pages, endpoints, DB rows). Persist the contract as a file.
4. Generator implements one feature at a time, commits after each, and
   produces a short self-eval before handoff. Self-eval is intentionally
   light — the load-bearing judgment lives in the Evaluator.
5. Evaluator drives the live application as a user would (clicking through
   flows, hitting endpoints, inspecting DB state) and grades against each
   criterion. Any criterion below threshold fails the sprint, regardless
   of average score — this prevents a feature with great craft and broken
   UX from passing.
6. Generator either refines against specific Evaluator findings or, if the
   sprint passes, moves to the next contract. The loop terminates per
   sprint, not per feature.
7. After every two or three sprints, audit the harness itself — see
   `harness-evolution`. The right scaffolding for sprint one may already
   be obsolete by sprint five.

## Rules: Do

- Separate Generator and Evaluator into different processes. Same agent
  judging its own output preserves the self-evaluation bias the
  architecture exists to defeat.
- Negotiate the sprint contract BEFORE any implementation code. Without
  it, Generator ships what it interpreted while Evaluator judges what it
  imagined.
- Apply hard thresholds per criterion, not an averaged score. Averaging
  hides catastrophic failure on one axis behind craft on another.
- Persist Planner output, sprint contracts, and Evaluator reports as files
  in the repo. Chat-only artifacts disappear between sessions.
- Calibrate the Evaluator with few-shot examples that show the score
  breakdowns you want, especially for subjective criteria.

## Rules: Don't

- Don't use the Generator's own self-eval as the gating signal for sprint
  acceptance. Lopopolo's axiom: "Code is free" applies — generating a
  fresh Evaluator pass is cheap; trusting the author of the patch is not.
- Don't let the Planner descend into implementation detail. A Planner that
  picks ORM versions has stolen the Generator's job and locked in errors.
- Don't run a Generator-only loop "for now, we'll add Evaluator later".
  The architecture is load-bearing as a whole; Generator alone produces
  confidently-mediocre output.
- Don't reuse last quarter's harness on a new model release without
  re-auditing — see `harness-evolution`.

## Expected Behavior

After applying the skill, the team has three named roles, a written sprint
contract per sprint, and a clear pass/fail signal at each sprint boundary.
Generator output stops being graded on its author's enthusiasm and starts
being graded against criteria that existed before code was written.

Subjective quality conversations move from "does this look good?" to "does
this hit the four criteria above their thresholds?" — and when it doesn't,
the Evaluator's report names the specific gap.

## Quality Gates

- Three roles exist as distinct processes (not three personas in one chat).
- Sprint contract is a file in the repo, dated, and references the spec.
- Evaluator interacts with the running system, not screenshots or code.
- Each criterion has an explicit threshold; sprint fails on any miss.
- Few-shot calibration examples exist in the Evaluator's prompt and are
  refined when the Evaluator's judgment diverges from a human spot-check.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs naturally with `harness-evolution` (re-audit after model upgrades) and
`harness-evaluator-optimizer-loop` (the single-LLM version of the same shape,
useful for tighter loops inside a sprint). Methodology phase: 30 (Skills/Agents)
— this is where the architecture choice is made for a project.

## Output Artifacts

- Spec file (Planner output) committed to the repo.
- One sprint contract file per sprint.
- Evaluator report per sprint (markdown with score per criterion + bug list).
- Optional: Evaluator prompt with calibration examples, also committed.

## Example Constraint Language

- Use "must" for: separating Generator and Evaluator into different
  processes, persisting contracts and reports as files, hard threshold
  per criterion.
- Use "should" for: few-shot calibration, file-based handoff between
  sprints, Evaluator interacting with live system.
- Use "may" for: adding a fourth role (e.g., Designer agent) once the
  three-role baseline is stable, dropping the Planner once the spec is
  effectively frozen.

## Troubleshooting

- **"Evaluator keeps rubber-stamping Generator's output"**: the Evaluator
  is under-tuned. Read its logs against a human spot-check, find where
  judgment diverged, and inject a few-shot example into its prompt. Expect
  several rounds before grading stabilizes.
- **"Sprints keep blowing past contract scope"**: the contract is too
  loose. Add specific endpoints, page paths, or DB invariants to the
  acceptance list. Vague contracts produce sprawling implementations.
- **"We don't have a binary success criterion for this work"**: this is
  the subjective-quality case. Encode principles as criteria with weights
  and few-shot breakdowns rather than skipping the Evaluator. "Best
  designs are museum quality" wording in the criteria changes the
  character of the output, not just the score.
- **"Three roles is overhead we don't need on this small task"**: correct.
  Use solo or `harness-evaluator-optimizer-loop` instead. This skill is
  for long-running work, not single sessions.

## Concrete Example

A four-week MVP build: a small DAW (digital audio workstation) in the
browser. Planner expands a four-sentence prompt into a spec naming
roughly sixteen target features and a stack (React + Vite + FastAPI +
SQLite). Generator runs in Claude Code; Evaluator is a subagent with
Playwright MCP. Before the first feature, Generator and Evaluator
negotiate sprint one: "done means login flow has 95% success rate
against the eval suite plus handles three edge cases (expired session,
malformed token, race on simultaneous logins)." Generator implements,
self-evals, hands off. Evaluator drives the live site, finds that the
expired-session path silently logs out without surfacing an error, fails
sprint one despite Generator's confident self-eval. Generator refines
against the specific finding. Sprint one passes, the team moves to
sprint two. Six hours and roughly $200 in tokens later, the core
features work end-to-end — measurably better than a solo run that
collapsed in twenty minutes.

## Sources

- `[[concepts/harness-engineering]]`
- `[[sources/harness-design-anthropic]]`
- Synthesized from Prithvi Rajasekaran's 2026 Anthropic engineering post
  "Harness design for long-running application development" via Danilo's
  wiki paraphrase. Sprint-contract framing extends the original article's
  Generator/Evaluator pairing with explicit pre-code negotiation.
