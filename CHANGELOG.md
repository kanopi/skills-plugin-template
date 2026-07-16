# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2026-07-16

### Added

- Initial template extracted from kanopi/cms-cultivator's plugin tooling
  as part of the 2026-07 repo split.
- `scripts/validate-frontmatter.sh` — frontmatter validation for skills,
  agents, and Codex openai.yaml policies (agent section skipped in
  agent-less repos).
- `scripts/run-evals.js` — TF-IDF trigger-routing eval with `--min-rank1`
  CI floor and description-collision detection at a configurable
  threshold (default 75%).
- `scripts/check-codex-parity.sh` — drift detection between
  `agents/*/AGENT.md` and `.codex/agents/*.toml` (bijection, name and
  description parity, required fields) plus openai.yaml policy checks.
- `scripts/package-plugin.sh` and `scripts/package-skills.sh` — packaging
  for Claude Desktop plugin and per-skill `.skill` uploads; plugin name
  derived from the manifest.
- `tests/test-plugin.bats` — generic scaffold with dynamic count parity
  (no hardcoded skill/agent counts) and foreign-namespace Task() gate.
- `.github/workflows/test.yml` — BATS, frontmatter validation, routing
  evals, codex parity, secret scan, JSON/YAML validation.
- `.github/workflows/release-artifacts.yml` — release artifact packaging.
- Example skill + agent pair so the template's own CI runs green
  end-to-end; downstream repos delete them.
