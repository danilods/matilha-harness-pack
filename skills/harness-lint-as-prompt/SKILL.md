---
name: harness-lint-as-prompt
description: Use when writing lint rules — embed remediation path in error messages, not just the symptom.
category: harness
version: "1.0.0"
requires: []
optional_companions: []
---

## When this fires

Use when the team is about to author a custom lint rule, or when an
existing lint is firing but agents (and humans) keep ignoring it. Fires
when an engineer writes a rule like "no await in loop" with a one-line
forbidden-marker message and assumes the rule will fix the behaviour.
The skill reframes lint output as a prompt the agent reads at a high-
leverage moment — and shows how to write that prompt so the next
keystroke is a fix, not an investigation.

## Preconditions

- The team has the technical ability to write custom lint rules (ESLint
  custom plugin, Ruff plugin, equivalent). Without that, the skill
  applies in spirit but not in mechanism.
- There is at least one canonical example in the codebase of the
  preferred pattern. A lint with no example to point at struggles to
  guide a fix.
- The lint will fire at a moment the agent or human can act — on save,
  on commit, on push. Lints that only fire in nightly batch lose most
  of their teaching power.

## Execution Workflow

1. Name the symptom and the remedy together before writing any rule.
   The symptom is what the rule detects; the remedy is what the author
   should do instead. If you cannot articulate the remedy in one
   sentence, the rule is not ready — get clearer on what the team
   wants, not just what it forbids.
2. Write the lint message with three parts in order: what is wrong,
   what to do instead, where to look for the canonical example. Keep
   it to two or three sentences — long messages get skimmed, short
   messages with the right shape get read.
3. Attribute the lint to a persona in the message or in a tag. A lint
   labelled "(reliability)" reads differently from one labelled
   "(frontend)" — the persona tells the agent which lens to apply
   when fixing.
4. Trigger only on the cases the team actually wants to forbid. A
   noisy lint trains the agent to ignore lints in general, which
   undoes the work of every other lint in the codebase. Tight
   triggers beat broad triggers.
5. Provide an escape-hatch syntax (eslint-disable-next-line with a
   required reason, or equivalent). The exception should require
   thought, not just a flag — the reason text becomes a comment the
   reviewer can audit.
6. Validate the message against an actual fix attempt. Have an agent
   (or a junior engineer) read only the lint output and try to fix
   the code without further context. If they cannot, the message is
   under-specified.
7. Add the rule, run it across the repo once to surface existing
   violations, and triage. Fix or grandfather; do not add a rule that
   leaves the repo with a permanent backlog of warnings — warnings
   become noise.

## Rules: Do

- Write every lint message as if it were a prompt to the next agent
  that will see the file. Because it is.
- Include the remediation path in the message body. Symptom alone
  forces investigation; symptom plus remedy enables a one-shot fix.
- Reference a canonical example by path or doc link when one exists.
  The agent can fetch the example and pattern-match without
  trial-and-error.
- Tag the lint with a persona so the agent knows which lens to apply.
- Re-read messages in the firing context periodically — a message
  that read well in the rule file may read badly when surrounded by
  twenty other lint hits.

## Rules: Don't

- Don't ship a lint with only a forbidden marker in the message ("await
  in loop forbidden"). The signal-without-remediation pattern is the
  most common reason lints get ignored.
- Don't write paragraphs in the lint message. Two or three sentences
  is the budget; anything more gets skimmed and the remedy is lost.
- Don't trigger on cases the team actually accepts. Each false positive
  costs trust in the entire lint set.
- Don't allow escape-hatch comments without a reason field. A bare
  disable line is invisible at review; a disable line with a sentence
  is auditable.

## Expected Behavior

After applying the skill, agent retry rates on lint failures drop. Most
fixes happen in one round because the message contained the remedy.
Human reviewers stop adding "see lint message" follow-up comments because
the lint message already said what the comment would have said.

Over time, the lint set becomes the team's shared muscle memory for
patterns. New patterns the team adopts get encoded as new lints with
the same shape; obsolete patterns get removed cleanly because each lint
has a documented intent.

## Quality Gates

- Every lint message names the remedy, not just the symptom.
- Every lint references a canonical example or a doc section, by path
  or link, where one exists.
- Persona attribution is present in the message or in a structured tag.
- The repo runs clean against the lint after introduction; no
  permanent backlog of warnings.
- Escape-hatch usage requires a reason field that survives to PR
  review.

## Companion Integration

No direct matilha-ux-pack or matilha-growth-pack sibling; this skill is foundational.
Pairs internally with `harness-nfrs-as-prompts` (the parent concept that
chooses lint as one surface among many) and `harness-test-source-code-structure`
(the structural cousin — when the rule is about code shape rather than
behaviour, prefer a test). Methodology phase: 30 (Skills/Agents) — lint
authoring is part of the skill scaffolding for the agent; also 50
(Qualidade) — lints are first-line quality enforcement.

## Output Artifacts

- Custom lint rule files committed to the repo.
- Updated lint configuration (eslint config, pyproject.toml, etc.) so
  the rule is active by default.
- Optional: a `LINTS.md` doc summarizing the team's custom lints, their
  intent, and their canonical examples.

## Example Constraint Language

- Use "must" for: remediation in the message body, persona attribution,
  reason text on escape-hatch comments.
- Use "should" for: linking to a canonical example, validating the
  message against a fix attempt before shipping.
- Use "may" for: the exact wording style (imperative vs declarative),
  whether to include a code-fix suggestion in the lint output.

## Troubleshooting

- **"The agent keeps fixing the lint by adding a disable comment"**:
  the disable does not require a reason, or the reason is not surfaced
  at review. Add a reason field and a review-time check that flags any
  disable without a reason longer than a few words.
- **"The lint fires too often"**: the trigger is broader than the
  team's actual rule. Tighten — false positives erode trust faster
  than false negatives.
- **"Agents fix the symptom but introduce a worse pattern"**: the
  remedy in the message points at a pattern the team does not
  actually want. Re-read the message; if the remedy was vague, the
  agent will pick the easiest legal alternative, which is often
  worse than the original.
- **"Engineers ignore the lint because it slows them down"**: the
  message reads as nagging rather than guidance. Rewrite it as
  helpful — "you have X, the team's choice is Y because Z" — and
  see whether perception changes before changing the rule itself.

## Concrete Example

A team adds a lint that fires on files over 350 lines. The first version
of the message reads: "File too large (max 350 lines)." Agents respond
by deleting comments, inlining helpers, or compressing whitespace —
none of which addresses the underlying problem. The team rewrites the
message: "File 412 lines exceeds 350. Split by responsibility — see
docs/DESIGN.md section file-size for the team's decomposition guide.
Suggestion: extract `parseRequestBody` and `validateAuthHeader` into
separate modules." Same rule, different prompt. Agent retry rate drops
from three attempts to one in 80% of cases, and the resulting splits
match the team's intent without further review feedback. The lint
becomes a teaching surface, not a tax.

## Sources

- `[[concepts/nfr-as-prompt]]`
- `[[sources/harness-engineering-lopopolo-openai]]`
- Synthesized from Ryan Lopopolo's 2026 AI Engineer London talk
  "Harness Engineering" via Danilo's wiki paraphrase. The
  symptom-vs-remedy distinction and the lint-as-prompt framing come
  directly from Lopopolo's keynote; the message-shape recipe (what,
  what-instead, canonical example) is the harness-pack
  operationalization of the same principle.
