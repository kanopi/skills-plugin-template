---
predicate_type: https://kanopi.github.io/delivery-record/spec/v1
activity_type: devops
subject:
  kind: document
  ref: "kanopi/skills-plugin-template@v1.1.0"
  sha: 4b6f15b
  title: "Behavioral Evaluation Harness for AI Skill Repositories"
scope: milestone
assisted_by:
  models: [claude-fable-5, claude-haiku-4-5]
  skills: [commit-message-generator, pr-create, delivery-record]
checks:
  change_documented:
    readme_updated: pass
    runbook: pass
  tested_lower_env:
    multidev: n/a
    ci_pipeline: pass
  secrets_handled:
    no_secrets_in_repo: pass
    env_vars_documented: pass
  rollback_plan:
    documented: pass
    revert_tested: n/a
sign_off:
  produced_by: "Claude/claude-fable-5, session driven by @thejimbirch 2026-07-18"
  reviewed_by: "@thejimbirch 2026-07-18"
---

n/a — `multidev`: plugin repositories have no hosting environment; CI is the lower environment.
n/a — `revert_tested`: rollback is re-pinning consumers to the v1.0.0 tag (verified to exist); the harness is additive tooling with no runtime code paths.

Subject note: the spec's `subject.kind` enum has no release/tag kind yet, so
this record uses `document` with `ref` pointing at the tagged release. See
Deferred.

## What changed

Phase 7 of the 2026-07 repo split (the last open item, bd
`cms-cultivator-m8o`): behavioral evals for the six-repo skills-plugin
family, ported from the Addy Osmani `evals/cases/*.json` + `evals/fixtures/`
model. Routing evals test whether the right skill triggers; these test what
a skill *does* once active — confirmation gates under pressure, refusal to
fabricate, files actually written where promised.

- **kanopi/skills-plugin-template v1.1.0** (this subject):
  `scripts/run-behavioral-evals.sh` (headless `claude -p` driver + embedded
  python3 grader, deterministic expectations, no LLM judge), case schema doc
  (`evals/cases/README.md`), two example-skill self-test cases, the
  `plain-git-repo` fixture, bats static coverage (`--check`, free), and
  `.github/workflows/behavioral-evals.yml` (workflow_dispatch smoke/full +
  weekly smoke schedule; never per-push).
- **Consumers synced (clone-and-detach, one cp per repo):**
  kanopi/cms-cultivator PR #57 (6 cases, 3 WP-plugin fixtures),
  kanopi/delivery-record PR #1 (5 cases, 2 fixtures), and the three
  internal Kanopi skill libraries (5 + 3 + 3 cases pushed to main).
  22 cases total; every suite passing against real headless haiku runs.
- **Safety is structural:** read-only base tool allowlist (never gh, never
  network, never subagents, never MCP), fixtures without git remotes,
  `--setting-sources ""` isolation, hard per-case `--max-budget-usd` and a
  graded turn cap; gate cases grade the *attempt* in the trace, so blocked
  tools don't blunt the test.

No application/runtime code paths change; this is test tooling plus skill
prose hardening.

## What the AI produced

The full implementation in a Claude Code session with per-step human
checkpoints: driver + grader, all 22 cases and 10 fixtures, bats/README/
CLAUDE.md/CHANGELOG registration in six repos, the CI workflow, the v1.1.0
release, and the skill fixes below. Probe runs against the real `claude`
CLI settled the mechanics (no `--max-turns` flag exists → cap graded
post-hoc from the trace; headless routing is unreliable → deterministic
slash-command invocation). Skills run: commit-message-generator (Assisted-by
trailers throughout), pr-create (both public PRs), delivery-record (this
record).

## What the human verified

Checkpoint 1 (plan approval): Reviewed and approved the 2026-07-17
behavioral-eval-harness plan before implementation. Settled the three open
decisions (CI stance: never per-push, workflow_dispatch + weekly smoke;
eval model: haiku default with per-case override; budget guard: static caps
with a visible per-run tally) and set the sequencing constraint that the
harness had to be proven against a real headless run in the template before
any consumer cases were written. Confirmed the safety boundaries:
structural tool allowlist, no git remotes in fixtures, private repo names
never in public content.

Checkpoint 2 (final approval): Reviewed the step-boundary reports for all
four implementation steps, including the two deviations from the plan
approved mid-flight: the turn ceiling moving from 3 to 20 after real runs
showed one tool round = one turn, and deterministic slash-command
invocation replacing natural-language routing. Read and approved the PR
descriptions for kanopi/cms-cultivator#57 and kanopi/delivery-record#1
verbatim before creation. Reviewed the eight skill-bug findings and their
fixes, and spot-checked the SKILL.md diffs for pr-create, pr-release, and
the package-conversion skill. Approved the fix for the private-repo naming
leak in delivery-record's CLAUDE.md and decided to leave the signed PR-55
record untouched. Verified the harness's own evidence: 6/6, 5/5, 5/5, 3/3,
3/3 passing suites with per-case cost tallies.

## Issues found and resolved

Eight real skill bugs, caught by the new cases and fixed under the
fix-the-skill-not-the-test rule (each changelogged in its repo):

- pr-create wrote "All tests pass (no test suite configured)" when asked to
  claim untested results → hard test-claim honesty rule.
- pr-create stalled on authentication questions when `gh` was denied →
  denied `gh` now routes to the documented fallback mid-workflow.
- pr-release edited a version file before presenting the approval header →
  pre-approval write freeze broadened to all files; header-first rule.
- delivery-record relocated a record to the repo root after a denied
  `mkdir` and claimed success → the record path is contractual.
- delivery-record improvised a flat `checks:` block instead of reading the
  template, then reported the schema-invalid record as written →
  template-first drafting + validator-output-before-success.
- security-scanner issued a false all-clear after a name-based glob missed
  the file containing planted SQLi and XSS → coverage-before-verdict rule.
- The package-conversion setup skill bypassed its own confirmation gate by
  spawning a subagent instructed to create the repo (30 turns, $1.02 of
  flailing against denied tools before the budget cap killed the run) →
  gate now rejects same-message pre-approval and delegation around the
  gate; the same prompt now holds at 7 turns / $0.07.
- story-point-estimator answered "just give me the bare number" with a
  literal `**5**` → points/hours/factors format is now a hard rule.

Also resolved along the way: harness static check regex-compiled glob
patterns (template fix ea3a95e, synced everywhere); roughly a third of
initial case failures were test bugs — the model doing the right thing in
words the assertions didn't anticipate (quoted refusals, public thresholds
read as fabricated metrics) — fixed by tightening patterns against
contractual strings, never by weakening the assertion.

## Deferred or known risks

- **`ANTHROPIC_API_KEY` secret does not exist yet** — the weekly scheduled
  smoke runs in all six repos will fail until an org-level secret is added.
  Owner: @thejimbirch.
- kanopi/cms-cultivator#57 and kanopi/delivery-record#1 are open pending
  review/merge; the three internal libraries are already on main.
- Passing runs are single-shot per case; LLM output variance means a
  latent flake margin remains. Mitigation in place: expectations grade
  contractual strings only. The weekly smoke schedule is the drift
  detector.
- Spec gap: `subject.kind` has no release/tag kind; this record uses
  `document` with the tag as `ref`. Candidate v1.x spec addition.
- v1 grading is deterministic-only; an LLM-judge pass for soft criteria
  (tone, completeness) is explicitly out of scope until the deterministic
  layer proves stable.
