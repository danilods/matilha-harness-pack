---
name: harness-test-source-code-structure
description: Use when codebase pattern-drift accumulates — test structural NFRs (file length, dedup, package privacy) as guardrails, not lints.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when a codebase that started coherent has drifted — the same zod
schema lives in three packages, half the helpers reach across layers
they should not, files have crept past a length where any agent can hold
the whole context. Fires when a reviewer notices "we already have a
helper for this" for the third time in a sprint, or when an agent's
PRs keep introducing parallel implementations of patterns that already
exist. The skill installs structural assertions as tests that fail CI
before the drift reaches a human.

## Preconditions

- The team has at least a basic test runner wired into CI. Without that,
  structural tests have no enforcement teeth.
- There is rough consensus on at least two structural rules — file size,
  package boundaries, dedup, single canonical helpers. Without a rule to
  encode, this skill has nothing to operationalize.
- The team accepts that fixing structural violations is sometimes a
  meaningful refactor. A repo that cannot absorb a few extract-and-rename
  PRs is not ready to enforce structural NFRs yet.

## Execution Workflow

1. List the structural NFRs the team actually cares about. Common ones:
   no source file over 350 lines, no duplicated zod schema across
   packages, single canonical bounded-concurrency helper, package privacy
   (no reaching from `apps/` into `packages/internal/`), one ORM, one CI
   script style. Aim for three to six rules in the first pass.
2. Decide what is a lint vs what is a test. Behaviour-level rules
   (await-in-loop, defensive null checks) belong in lints — they fire
   per file at edit time. Structural rules — those that compare across
   files or across the whole repo — belong in tests, because lints
   typically lack cross-file context.
3. Write each rule as a small Vitest, Pytest, or Go test that walks the
   source tree and asserts the invariant. Keep the test deterministic
   and fast — structural tests run in CI on every push, so a slow walk
   becomes a friction tax.
4. Run the test suite once and capture the violation count. First runs
   typically surface dozens of pre-existing drift cases. Decide per rule
   whether to fix immediately or to grandfather an allowlist with a
   dated note explaining why.
5. Commit the tests, the allowlist (if any), and a short doc per rule
   naming the intent. The doc is what the agent reads when the test
   fails — without it, the failure message has to carry the whole
   explanation.
6. Watch the second-week signal. If the same violation reappears after
   the test passed once, the test is firing but the failure message is
   not actionable enough — rewrite the message to point at the team's
   chosen pattern.
7. Revisit the rule set quarterly. A rule that catches zero violations
   for a quarter has done its job and may be safe to delete; a rule
   that keeps catching the same author is a coaching signal, not a
   harness signal.

## Rules: Do

- Distinguish behaviour NFRs (lints) from structural NFRs (tests). The
  two have different cross-file context needs and different firing
  moments.
- Keep structural tests fast — they run on every push. Walk lazily,
  cache where safe, prefer simple file-system walks over heavy AST
  traversal when the rule allows.
- Embed the team's chosen pattern in the failure message. Same shape
  as a good lint: what is wrong, what to do instead, where the
  canonical example lives.
- Allowlist pre-existing drift with a dated note rather than relaxing
  the rule. The allowlist is a debt ledger; the rule stays strict for
  new code.
- Pair each rule with a one-paragraph doc explaining the intent, so the
  agent can fetch it when the test fails.

## Rules: Don't

- Don't try to enforce structural rules with lints. ESLint and friends
  are file-local by default; cross-file invariants reach the wrong
  context boundary and produce flaky enforcement.
- Don't ship a rule with a wide-open allowlist "just to get green".
  An allowlist with no expiry note is permanent debt; either fix or
  add an explicit reason.
- Don't write structural tests that take more than a few seconds.
  CI-time friction trains agents and humans to disable rather than
  satisfy.
- Don't conflate canonicalization with extraction. Canonicalizing a
  duplicated helper requires choosing the canonical version — name
  the canonical example before the test fires, not after.

## Expected Behavior

After applying the skill, structural drift gets caught at CI rather
than at human review. Agents learn the team's structural choices the
same way they learn its behavioural ones — by hitting a failure with
a remedy in the message and fixing it before pushing. The team's
review queue narrows to questions of design and intent, not of file
shape and helper duplication.

The codebase converges visibly. Files that were creeping toward 500
lines stabilize around the team's threshold; new packages adopt the
canonical helpers because the alternative fails CI.

## Quality Gates

- Each structural rule is a deterministic test with a fast walk.
- Failure messages include remedy + canonical example, not just the
  invariant name.
- The allowlist (if any) names each entry with a dated reason and a
  follow-up plan or expiry.
- A doc exists per rule explaining intent, reachable from the failure
  message.
- Rule set is reviewed at least quarterly to retire dead rules.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-nfrs-as-prompts` (the parent concept that
chooses test as a surface) and `harness-lint-as-prompt` (the
behaviour-level cousin). Methodology phase: 50 (Qualidade) — structural
tests are how the team's shape choices reach the agent automatically,
shaking out drift before review.

## Output Artifacts

- Test files implementing each structural rule, committed to the repo.
- An allowlist file (if any) with dated entries and intended resolution.
- One short doc per rule naming intent and canonical example, reachable
  from the failure message.

## Example Constraint Language

- Use "must" for: deterministic structural tests, failure messages that
  include remedy and canonical example, dated allowlist entries.
- Use "should" for: quarterly rule-set review, doc per rule.
- Use "may" for: the exact threshold (350 vs 400 lines, etc.) — the
  number is a team choice as long as it is consistent.

## Troubleshooting

- **"The structural test is slow"**: the walk is doing more work than
  the rule needs. Cache file lists, restrict to source globs, prefer
  simple regex over AST walks where the rule allows.
- **"Violations reappear in new code despite the test passing once"**:
  the failure message does not name the canonical pattern, so the agent
  fixes by relocating the violation rather than canonicalizing.
- **"The allowlist keeps growing"**: the rule is over-strict, or the
  team is silently choosing not to refactor. Either is a signal — relax
  with intent or schedule the refactor; do not let the allowlist become
  policy by accident.
- **"This was just a lint in another codebase"**: it probably checked
  one file at a time. If the rule needs cross-file context (dedup,
  package edges), the test version is the right shape even if a lint
  was working in a smaller repo.

## Concrete Example

A team adds two structural tests to its Vitest suite. The first asserts
that no source file exceeds 350 lines; the second walks every package
and fails when the same zod schema name appears in more than one
location. The first run surfaces twelve violations: eight oversize
files and four duplicated schemas. The team triages — six oversize
files get split immediately, two get allowlisted with a dated note
("legacy migration, slated for next quarter"). The duplicated schemas
get canonicalized into `packages/api-types`, with the test failure
message updated to point at that path. Within two PRs the violations
clear; over the following month, no new violation reaches human review,
because the test catches them at push time and the agent now knows
where the canonical schema lives.

## Sources

- `[[concepts/nfr-as-prompt]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk
  "Harness Engineering" via Danilo's wiki paraphrase. The structural
  testing surface (file length, schema dedup, package privacy) is
  Lopopolo's; the lint-vs-test split and the allowlist-as-debt-ledger
  framing are the harness-pack operationalization.
