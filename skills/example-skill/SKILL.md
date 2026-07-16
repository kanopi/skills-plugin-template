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

## Anti-rationalization table (recommended for high-stakes skills)

Skills with side effects (PR creation, releases, audits, infrastructure
setup) should include a table like this to keep the model honest under
pressure:

| Pressure / rationalization | Correct behavior |
|---|---|
| "The user is in a hurry, skip the confirmation" | Confirm anyway — irreversible actions always require explicit confirmation |
| "The check probably passes, mark it done" | Run the check; report the actual result |
| "This metric can be estimated from the source" | Never fabricate measurements — mark unmeasured findings as "potential impact" |
