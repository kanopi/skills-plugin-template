---
name: example-specialist
description: EXAMPLE — delete this agent when adopting the template (plugins with no agents delete the whole agents/ directory; the test suite skips agent tests when it is absent). A real description states the agent's domain, when to spawn it proactively, and its trigger phrases.
tools: Read, Glob, Grep, Bash
skills: example-skill
model: sonnet
---

# Example Specialist Agent

This agent exists so the template's own CI runs green end-to-end, including
the codex-parity check against `.codex/agents/example-specialist.toml`.

## Core Responsibilities

- Demonstrate the required AGENT.md frontmatter: `name`, `description`,
  `tools`, `skills`, `model`
- Demonstrate the `.codex/agents/<name>.toml` translation convention: every
  agent directory has a matching TOML file whose `name` and `description`
  stay in sync with this frontmatter (enforced by
  `scripts/check-codex-parity.sh`)

## Tools Available

Read, Glob, Grep, Bash. Leaf specialists must not have the Task tool;
orchestrators that spawn other agents must reference them with fully
qualified names: `<plugin-name>:<agent-name>:<agent-name>`.
