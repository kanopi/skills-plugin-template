# Claude Context for the Skills Plugin Template

This repo is the shared scaffold for Kanopi's Claude Code skills plugins.
It is cloned and detached per plugin — changes here do **not** propagate
automatically; improvements must be ported to downstream repos manually.

## Invariants

- **Skills are the single source of truth.** `.codex/agents/*.toml` and
  `skills/*/agents/openai.yaml` are translations; `scripts/check-codex-parity.sh`
  enforces name/description parity.
- **No hardcoded counts** of things that grow (skills, agents) in docs or
  tests — use dynamic parity checks (`tests/test-plugin.bats` shows the
  pattern: directory count vs `### N.` entries in `skills/README.md`).
- **Manifest parity:** `.claude-plugin/plugin.json` and
  `.codex-plugin/plugin.json` share the same `name` and `version`.
- **Fully qualified agent references:** `Task(<plugin>:<agent>:<agent>)`
  must use this plugin's own name; foreign prefixes fail the bats suite.
- The `example-skill` / `example-specialist` pair exists only so the
  template's own CI runs green — downstream repos delete them.

- **Behavioral evals are deterministic and off the push path.**
  `evals/cases/*.json` + `evals/fixtures/` + `scripts/run-behavioral-evals.sh`
  test skill behavior (gates, refusals, honesty) via headless `claude -p`
  runs. Expectations grade contractual strings the skill itself mandates —
  if a skill lacks one, add it to the skill first. A failing case is a
  skill bug: fix the skill, not the test, and changelog it.

## When adding a skill (downstream repos)

1. Create `skills/<name>/SKILL.md` with `name` + `description` frontmatter
   (trigger phrases, use case, scope boundaries).
2. Append the next `### N.` entry to `skills/README.md` (never insert
   mid-list — numbering parity is tested).
3. Add 2–5 routing prompts to `evals/routing-prompts.json`.
4. Add a `CHANGELOG.md` entry under `[Unreleased]`.
5. Skills with side effects (PR creation, releases, file writes, infra
   changes): add at least one gate case and one pressure case to
   `evals/cases/` (schema: `evals/cases/README.md`), then run
   `./scripts/run-behavioral-evals.sh --case <name>` for each.
6. Run the verification quartet:
   `validate-frontmatter.sh`, `bats tests/test-plugin.bats`,
   `run-evals.js --min-rank1 75`, `check-codex-parity.sh`.
   (Bats includes the behavioral cases' static `--check` — free, no API
   calls.)

## When adding an agent (downstream repos)

1. Create `agents/<name>/AGENT.md` with `name`, `description`, `tools`,
   `skills`, `model` frontmatter.
2. Create `.codex/agents/<name>.toml` with matching `name`/`description`
   (whitespace-normalized parity is machine-checked), `model`, and
   `developer_instructions` translated from the AGENT.md body.
3. Leaf specialists must not have the Task tool; orchestrators must.
