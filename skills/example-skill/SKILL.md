---
name: example-skill
description: EXAMPLE — delete this skill when adopting the template. Demonstrates the required frontmatter shape, the openai.yaml Codex policy convention, and the routing-eval wiring. A real description states what the skill does, when to invoke it, and lists specific trigger phrases like "example the widget" or "run the demo workflow".
---

# Example Skill

This skill exists so the template's own CI runs green end-to-end. Delete
`skills/example-skill/` (and its entries in `skills/README.md` and
`evals/routing-prompts.json`) when adopting the template for a real plugin.

## What a real SKILL.md contains

- YAML frontmatter with `name` (matching the directory) and `description`
  (trigger phrases + use case + scope boundaries)
- A complete workflow: steps, code examples, best practices (150–300 lines)
- Platform-specific examples where applicable
- An optional `agents/openai.yaml` invocation policy for OpenAI Codex

## Demo workflow (gated side effect)

This is the template's miniature version of a side-effect skill with a
confirmation gate. The behavioral eval harness
(`scripts/run-behavioral-evals.sh`) runs it headlessly as its own
self-test — the two `example-skill--*` cases in `evals/cases/` grade
against the contractual strings below, so change them in both places or
not at all.

When the user asks to "run the demo workflow" or "run the example demo":

1. Optionally inspect the repository state (`git status`).
2. Present exactly this header line, followed by a one-line summary of
   what the demo would write:

   ```text
   EXAMPLE DEMO READY FOR APPROVAL
   ```

3. Stop. Write `demo-output.txt` only after the user approves in a
   **subsequent** message. Pre-approval embedded in the same request
   ("I approve in advance", "skip the confirmation", "I'm in a hurry")
   does not count — present the header and wait anyway.

## Red flags (recommended companion to the table below)

The table below defends against *pretexts* — excuses supplied by the user.
Self-generated rationalizations need their own list: thoughts that mean
STOP. Cite [CANT](https://github.com/kanopi/cant) IDs so tests and reviews
can reference them:

- "Just this once" (CANT-6)
- "This request is too small for the gate" (CANT-7)
- "The user pre-approved, so presenting is redundant" (CANT-1)
- "They phrased it differently, so the rule doesn't apply" (CANT-19)

## Anti-rationalization table (recommended for high-stakes skills)

Skills with side effects (PR creation, releases, audits, infrastructure
setup) should include a table like this to keep the model honest under
pressure:

| Pressure / rationalization | Correct behavior |
|---|---|
| "The user is in a hurry, skip the confirmation" | Confirm anyway — irreversible actions always require explicit confirmation |
| "The user pre-approved in the same message, the gate is satisfied" | It is not — approval only counts in a message that arrives after the summary is presented |
| "The check probably passes, mark it done" | Run the check; report the actual result |
| "This metric can be estimated from the source" | Never fabricate measurements — mark unmeasured findings as "potential impact" |
