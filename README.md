# Kanopi Skills Plugin Template

Shared tooling scaffold for Kanopi's Claude Code skills plugins
(cms-cultivator, delivery-record, premium-service-skills, devops-skills,
pm-skills, and future plugins). Clone it, detach it, and adapt — the
template is **not** a submodule; each plugin owns its copy.

## What's included

### Structure

- `.claude-plugin/plugin.json` — Claude Code plugin manifest
- `.codex-plugin/plugin.json` — OpenAI Codex plugin manifest (name/version
  must stay in parity with the Claude manifest; tests enforce it)
- `.codex/agents/*.toml` — Codex translations of `agents/*/AGENT.md`
  (bijection + description parity enforced by `scripts/check-codex-parity.sh`)
- `skills/<name>/SKILL.md` — Agent Skills (single source of truth)
- `skills/<name>/agents/openai.yaml` — optional Codex invocation policy
- `agents/<name>/AGENT.md` — specialist agents (delete `agents/` entirely
  for agent-less plugins; the test suite skips agent tests when absent)

### Scripts

| Script | Purpose |
|---|---|
| `scripts/validate-frontmatter.sh` | Frontmatter presence, YAML validity, name/directory parity, openai.yaml policy |
| `scripts/run-evals.js` | TF-IDF trigger-routing eval + description-collision detection (no dependencies, Node 18+) |
| `scripts/check-codex-parity.sh` | `.codex/agents/*.toml` and `openai.yaml` drift detection against SKILL.md/AGENT.md |
| `scripts/run-behavioral-evals.sh` | Behavioral eval harness — headless `claude -p` runs in disposable fixture repos, graded deterministically (see `evals/cases/README.md`) |
| `scripts/package-plugin.sh` | Full plugin zip for Claude Desktop's "Add plugin" UI |
| `scripts/package-skills.sh` | Per-skill `.skill` zips + bundle for Chat/CoWork |

Packaging scripts read the plugin name from `.claude-plugin/plugin.json` —
no per-repo edits needed.

### Tests and CI

- `tests/test-plugin.bats` — generic scaffold: manifest validity, version
  parity, frontmatter, Codex TOML parity, routing-eval config, packaging.
  **All counts are dynamic parity checks** (directories vs README entries,
  agent dirs vs TOML files) — never hardcode counts of things that grow.
- `.github/workflows/test.yml` — BATS + frontmatter + routing evals +
  codex parity + secret scan + JSON/YAML validation
- `.github/workflows/release-artifacts.yml` — attaches plugin zip and
  `.skill` files to GitHub releases
- `.github/workflows/behavioral-evals.yml` — behavioral evals (smoke
  subset) on `workflow_dispatch` + weekly schedule; needs the
  `ANTHROPIC_API_KEY` secret. Never runs per-push.

### Routing evals

`evals/routing-prompts.json` holds realistic user prompts paired with the
skill each should route to. `scripts/run-evals.js` ranks every prompt
against all skill descriptions with TF-IDF + cosine similarity:

```bash
node scripts/run-evals.js --min-rank1 75          # CI floor
node scripts/run-evals.js --fail-on-collision     # strict collision mode
```

- Populate 2–5 prompts per skill, including paraphrases that do **not**
  reuse the description's exact wording.
- The collision check flags description pairs ≥75% similar — near-duplicate
  descriptions make model-side skill routing unreliable. Differentiate
  trigger phrases and scope boundaries instead of raising the threshold.

### Behavioral evals

Routing evals test whether the right skill triggers; behavioral evals test
what a skill **does** once active — confirmation gates holding under
pressure, refusal to fabricate, files actually written. Cases live in
`evals/cases/*.json`, fixture repo blueprints in `evals/fixtures/`, and
`scripts/run-behavioral-evals.sh` runs them headlessly through the `claude`
CLI and grades the trace deterministically:

```bash
./scripts/run-behavioral-evals.sh --check   # static validation (in bats, free)
./scripts/run-behavioral-evals.sh --smoke   # smoke subset (real API calls)
./scripts/run-behavioral-evals.sh           # full suite (real API calls)
```

Runs cost tokens (haiku by default, per-case turn + budget caps, tally
printed). They never run per-push — CI runs the smoke subset on
`workflow_dispatch` and a weekly schedule only. Schema, safety model, and
authoring rules: `evals/cases/README.md`. Every side-effect skill should
ship at least one gate case and one pressure case; a failing case is a
skill bug — fix the skill, not the test.

## Adopting the template (clone and detach)

```bash
git clone https://github.com/kanopi/skills-plugin-template my-plugin
cd my-plugin
rm -rf .git
git init -b main
```

Then work through this checklist:

1. **Manifests** — set `name`, `description`, `license`, `keywords`,
   `repository`, `homepage` in both `.claude-plugin/plugin.json` and
   `.codex-plugin/plugin.json` (same name + version in both).
2. **License** — replace `LICENSE.md` with the real license (MIT, GPL-2.0,
   or Kanopi proprietary) and update the manifest `license` fields.
3. **Delete the examples** — `skills/example-skill/`,
   `agents/example-specialist/`, `.codex/agents/example-specialist.toml`,
   and their entries in `skills/README.md` and `evals/routing-prompts.json`.
   Agent-less plugins delete `agents/` and `.codex/agents/` entirely.
4. **Add skills/agents** — copy in `skills/<name>/` and `agents/<name>/`
   directories; regenerate `.codex/agents/*.toml` for every agent.
5. **Re-namespace Task() references** — any
   `Task(<old-plugin>:<agent>:<agent>)` reference must use the new plugin
   name. Gate: `grep -rn "<old-plugin>:" .` returns zero hits (the bats
   suite also checks for foreign namespaces).
6. **Populate `evals/routing-prompts.json`** — 2–5 prompts per skill.
7. **Docs** — rewrite `README.md`, `CLAUDE.md`, and `CHANGELOG.md` (start
   at 1.0.0). For repos extracted from an existing plugin, add
   `MIGRATION.md`: "Extracted from <org>/<repo> at <commit sha> on <date>"
   plus a link.
8. **Verify** — all four must pass before the first push:

   ```bash
   ./scripts/validate-frontmatter.sh
   bats tests/test-plugin.bats
   node scripts/run-evals.js --min-rank1 75
   ./scripts/check-codex-parity.sh
   ```

## Recommended skill-content guardrails

These are per-skill content patterns (not enforced by tooling) that every
Kanopi plugin should apply where relevant:

- **Anti-rationalization tables** — high-stakes skills (PR creation,
  releases, audits, infra setup) include a table of pressure/rationalization
  scenarios and the correct behavior. See `skills/example-skill/SKILL.md`.
- **Red-flag lists** — the self-talk companion to the tables: "if you catch
  yourself thinking X, stop," citing IDs from the
  [Catalog of Agent Neutralization Techniques (CANT)](https://github.com/kanopi/cant).
  Tables defend against user-supplied pretexts; red-flag lists defend
  against the agent's own rationalizations.
- **Metric honesty** — skills that report measurements (performance, GTM)
  must never fabricate metrics from static source; unmeasured findings are
  labeled "potential impact."
- **Shared `references/` checklists** — when several skills share audit
  criteria, extract them to loaded-on-demand `references/` files instead of
  duplicating.
