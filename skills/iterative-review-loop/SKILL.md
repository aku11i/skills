---
name: iterative-review-loop
description: Iteratively review and fix implementation changes using implementation-review until no actionable Must or Should findings remain, committing fixes when working on an already committed branch.
---

# Iterative Review Loop

Use this skill when the user wants review and fixes repeated until the implementation has no actionable review findings.

## Required Skills

- Use `$implementation-review` for every review pass.
- Use `$gh-commit-and-push` only when the user asks to push, or when the surrounding task explicitly requires publishing the branch.

## Workflow

1. Inspect the repository state.
   - Run `git status --short --branch`.
   - Determine whether the target is local uncommitted work or an already committed branch/PR.
   - Gather the implementation purpose and background from the PR description, issue references, user request, branch name, commits, and diff. Keep this as the review brief for the full loop.
   - Preserve unrelated user changes. Stage or edit only files that belong to accepted fixes.

2. Run a review pass with `$implementation-review`.
   - Review the same target the implementation-review skill selects: uncommitted changes first, otherwise the current branch PR or branch diff.
   - Pass the review brief into each review pass so `$implementation-review` and any review subagents receive PR-overview-equivalent purpose, background, constraints, and non-goals.
   - Keep the findings in `Must` and `Should` terms. Ignore nit-level comments.

3. Decide what to do with each finding.
   - Fix findings that are correct, relevant to the change intent, and practical to address in the current scope.
   - Skip findings that are incorrect, already handled, outside the requested scope, or not worth changing for the PR purpose.
   - When skipping a `Must` or `Should`, record the reason clearly for the final report.
   - If a finding is ambiguous, inspect the code and intent before deciding. Ask the user only when the decision depends on product or policy context you cannot infer.

4. Implement accepted fixes.
   - Make the smallest change that resolves the finding.
   - Add or adjust focused validation when the finding involves behavior, regression risk, or reviewability.
   - Do not reformat or refactor unrelated code.

5. Validate the fixes.
   - Run the cheapest relevant tests, type checks, linters, or dry runs.
   - If validation is unavailable, risky, slow, or blocked by credentials, state that explicitly.

6. Commit when appropriate.
   - Treat the target as an already committed branch when it has a current PR, an upstream branch, or existing branch commits being reviewed.
   - If the target is an already committed branch, commit accepted fixes after validation and before the next review pass.
   - Inspect recent commit style before choosing the message.
   - If the target is only uncommitted local work, leave changes uncommitted unless the user asked for commits.

7. Repeat.
   - Run `$implementation-review` again after every accepted fix batch.
   - Reuse the same review brief, updating it only when the accepted fixes materially change the scope, constraints, or risk profile.
   - Continue until the latest review pass has no findings that require action.
   - Stop early only if the remaining findings are intentionally skipped, require user input, or cannot be fixed in the current environment.

## Completion Report

When the loop finishes, report:

- The final review result: no actionable findings remain, or list the remaining skipped/blocking findings with reasons.
- Fixes made, grouped by review pass if that helps.
- Commits created, if any.
- Validation commands and outcomes, including gaps.

Keep the report concise. Do not include nitpicks or unrelated observations.
