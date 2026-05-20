---
name: gh-approve-dependabot-prs
description: Review Dependabot pull requests one by one, investigate CI failures, assess dependency-update impact, comment on risky PRs, and approve only Dependabot PRs whose CI and impact review are acceptable.
---

# Approve Dependabot PRs

Use this skill to triage open Dependabot pull requests and approve only safe dependency updates.

## Workflow

1. Identify Dependabot pull requests.
   - Work in the target repository.
   - List open PRs authored by Dependabot, for example with `gh pr list --author "app/dependabot" --state open --json number,title,url,headRefName,baseRefName,statusCheckRollup`.
   - If the repository uses a different Dependabot author value, confirm it from the PR list before continuing.
   - Process one PR at a time. Do not batch approve.

2. Understand the PR before judging it.
   - Read the PR title, body, changed files, commits, and dependency metadata.
   - Inspect the diff with `gh pr diff <number>` or an equivalent base comparison.
   - Identify the dependency name, current version, target version, package manager, lockfile changes, and whether the update is direct or transitive.
   - Check release notes, changelog, security advisory, or migration notes when the diff does not fully explain the change.

3. Check CI first.
   - Inspect the PR checks with `gh pr checks <number>` or `statusCheckRollup`.
   - If checks are pending, skipped unexpectedly, missing, or stale, do not approve. Comment with the current check state and move to the next PR unless the user asked to wait or rerun checks.
   - If any required or relevant check failed, investigate the failure enough to be useful:
     - Read the failed job name, failing command, error output, and affected file or test when available.
     - Distinguish dependency incompatibility from environmental, flaky, or unrelated failures when evidence supports it.
     - Leave a PR comment summarizing the failure cause, supporting evidence, and suggested next action.
     - Move to the next PR without approving.

4. Assess impact when CI passes.
   - Determine how the updated dependency is used in the project:
     - Search imports, package references, build config, runtime config, generated files, lockfiles, Docker or CI config, and deployment manifests as relevant.
     - Check public API, CLI behavior, web or mobile UI behavior, server runtime behavior, build output, test tooling, and infrastructure paths touched by the dependency.
   - Look specifically for regression risks:
     - Breaking changes, removed APIs, changed defaults, changed peer dependency requirements, Node/runtime version changes, security model changes, performance regressions, data format changes, bundling changes, and platform support changes.
     - Lockfile churn that pulls in unrelated major upgrades or swaps package sources unexpectedly.
     - Test-only dependency changes that can still alter linting, formatting, type checking, snapshot output, generated code, or CI behavior.

5. Comment instead of approving when risk remains.
   - If the impact review finds a likely regression, incompatibility, unreviewable broad change, or missing validation for a high-risk path, leave a concise PR comment.
   - Include the concrete risk, where it appears in the diff or dependency notes, and what should be checked or changed before approval.
   - Move to the next PR after commenting.

6. Approve safe PRs.
   - Approve only when CI is passing and the impact review finds no concrete regression risk.
   - Use `gh pr review <number> --approve --body "<summary>"`.
   - Keep the approval body factual: mention passing CI and the impact areas checked.
   - Do not merge unless the user explicitly asks.

7. Report results after the pass.
   - List each PR processed with one outcome: approved, commented for CI failure, commented for impact risk, skipped for pending or missing checks, or skipped for missing context.
   - Include comments or approval URLs when available.
   - State any checks, release notes, or impact areas that could not be inspected.

## Comment Guidance

Use short, evidence-based PR comments. Avoid speculative wording when the evidence is incomplete.

For CI failures:

```text
I am not approving this yet because CI is failing.

Observed failure: <job and failing command/test>.
Evidence: <short error summary or relevant log line>.
Suggested next step: <rerun if flaky, update code/config, or investigate incompatibility>.
```

For impact risks:

```text
I am not approving this yet because the dependency update may affect <area>.

Risk: <specific breaking change or regression scenario>.
Evidence: <diff path, usage path, release note, or changelog item>.
Suggested next step: <validation or code change needed before approval>.
```

For approvals:

```text
CI is passing, and I checked <impact areas>. I did not find a concrete regression risk from this dependency update.
```

## Safety Rules

- Never approve a PR that is not authored by Dependabot.
- Never approve when required or relevant CI is failing, pending, missing, or stale.
- Never approve only from the PR title. Inspect the diff and the dependency's project impact.
- Do not dismiss broad lockfile changes as safe without checking what changed.
- Do not post duplicate comments if the same investigation result has already been commented.
- Preserve unrelated local changes. This workflow should not require editing the target repository.
- Do not merge, close, rerun CI, or push commits unless the user explicitly asks.

## Useful Commands

```bash
gh pr list --author "app/dependabot" --state open --json number,title,url,headRefName,baseRefName,statusCheckRollup
gh pr view <number> --json number,title,body,author,baseRefName,headRefName,commits,files,labels,url,statusCheckRollup
gh pr diff <number>
gh pr checks <number>
gh run view <run-id> --log-failed
gh pr comment <number> --body "<body>"
gh pr review <number> --approve --body "<body>"
```
