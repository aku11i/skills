---
name: implementation-review
description: Review implementation changes for application or infrastructure repositories. Use when asked to review code, inspect a pull request, evaluate uncommitted changes, or report Must and Should findings for web apps, command-line apps, mobile apps, backend services, or infrastructure changes.
---

# Implementation Review

Use this skill to review implementation changes and report only issues that are important enough to fix before or soon after merging.

## Review Target

1. Inspect repository state first.
   - Run `git status --short --branch`.
   - If the worktree has uncommitted changes, review those changes by default.
   - Include staged and unstaged changes. Check untracked files when they appear relevant.

2. If there are no uncommitted changes, review the current branch pull request.
   - Prefer `gh pr view --json number,title,body,baseRefName,headRefName,url` and `gh pr diff`.
   - If GitHub CLI is unavailable, compare the current branch with the likely base branch.
   - Do not invent PR metadata. State when PR context could not be loaded.

3. Preserve unrelated user work.
   - Do not modify files unless the user explicitly asks for fixes.
   - Do not revert, reformat, or stage changes during review.

## Review Workflow

1. Understand intent and scope.
   - Read the PR description, issue references, commit messages, changed files, and nearby code.
   - Identify the product behavior, data model, API contract, CLI behavior, mobile behavior, or infrastructure behavior the change is meant to affect.
   - Build a short review brief equivalent to a PR overview. Include the implementation purpose, background, user-visible or operational intent, important constraints, and any explicit non-goals or risks. If reviewing uncommitted local work without a PR description, infer this from the user's request, branch name, commits, issue references, and diff; clearly mark missing or inferred context.

2. Use subagents when the environment supports them.
   - Ask one or more subagents to review the implementation before producing the final findings.
   - Split subagents by broad review areas when that is useful, such as correctness and compatibility, security and data integrity, tests and validation, or operations and rollout.
   - Give each subagent the review target, the review brief, relevant diff context, and the same Must/Should severity rules. The review brief must carry the same purpose and background information a reviewer would normally get from the PR overview.
   - Ask each subagent to judge findings against that purpose and background, then report only concrete findings with file and line references.
   - Do not let subagent output bypass judgment. Reconcile duplicates, verify claims against the code when practical, discard unsupported or nit-level comments, and synthesize the final review yourself.
   - If subagents are unavailable, continue with a direct review and state no special note unless it affects confidence or validation.

3. Inspect the diff and the surrounding implementation.
   - Use `git diff --stat`, `git diff`, and focused file reads.
   - Trace call sites, configuration, data flow, permissions, migrations, feature flags, and error paths as needed.
   - For infrastructure, inspect plan-like changes, dependencies, permissions, rollout behavior, and rollback implications.

4. Validate where practical.
   - Run focused tests, type checks, linters, or dry-run commands when they are cheap and relevant.
   - If validation is risky, slow, unavailable, or requires missing credentials, state the gap.

5. Report findings first.
   - Prioritize concrete risks over style preferences.
   - Include file and line references whenever possible.
   - Explain the scenario, impact, and expected correction.
   - Avoid nitpicks, personal preference, broad rewrites, and speculative concerns without a plausible failure mode.

## Review Areas

Use these areas to decide what to inspect. Do not report every checklist item; report only concrete Must or Should findings.

- Requirements fit: the implementation matches the stated goal, issue, product behavior, CLI contract, mobile behavior, API contract, or infrastructure intent without missing required cases or adding unintended behavior.
- Correctness: normal paths, boundary values, empty and null inputs, invalid input, retries, timeouts, concurrency, ordering, state transitions, idempotency, and error handling behave as expected.
- Impact and compatibility: existing features, public APIs, schemas, migrations, configuration, saved data, integrations, generated files, and deployment behavior remain compatible or have a safe migration path.
- Security and privacy: authentication, authorization, tenant isolation, input validation, output escaping, dependency risk, secret handling, logging of sensitive data, and permission scope are appropriate for the change.
- Data integrity: writes are atomic enough for the domain, transactions and rollbacks are used where needed, partial failures are handled, and migrations avoid data loss or inconsistent state.
- Design and maintainability: responsibilities are clear, abstractions match existing project patterns, coupling is reasonable, names are accurate, and future changes will not require touching many unrelated places.
- Tests and validation: high-risk behavior has focused tests or equivalent validation, regression risk is covered, fixtures are realistic, and test changes actually assert the behavior being introduced.
- Performance and resource use: queries, loops, network calls, memory use, mobile responsiveness, startup cost, caching, and infrastructure scale are acceptable for realistic data volume and traffic.
- Operations and observability: failures produce actionable errors, logs, metrics, traces, alerts, runbooks, rollback behavior, and feature flags are sufficient for production diagnosis and recovery.
- Documentation and rollout: maintainers, users, operators, and downstream callers have the needed migration notes, configuration examples, changelog entries, or usage docs.

## Severity Rules

### Must

Use `Must` only for issues that should block merge because the change is wrong, unsafe, or not reviewable enough to trust.

Examples:
- The implementation does not satisfy the stated requirement.
- A clear bug breaks expected behavior, edge cases, state transitions, or concurrency.
- The change can corrupt, lose, leak, or inconsistently migrate data.
- Authentication, authorization, input validation, secret handling, or dependency security is unsafe.
- Existing behavior, public APIs, schemas, configuration, or compatibility are broken without a safe migration path.
- Required tests or validation are missing for high-risk behavior, making correctness impossible to judge.
- The change fails tests or introduces build, packaging, deployment, or startup failures.
- Operational recovery or investigation is materially blocked for a production-impacting path.

### Should

Use `Should` for issues that do not clearly block merge but are worth fixing in the same PR when practical.

Examples:
- The implementation diverges from established local patterns without a clear benefit.
- Responsibilities are blurred in a way that will make near-term changes harder.
- The code is unnecessarily complex or easy to misunderstand.
- Duplication creates likely future drift or bugs.
- Naming, structure, or API shape is misleading.
- Tests cover the happy path but miss important nearby cases.
- Logs, error messages, metrics, or user-facing failures are insufficient for practical diagnosis.
- Performance, scalability, resource usage, or mobile responsiveness may degrade under realistic load but is not clearly broken.
- Documentation, migration notes, or configuration examples are needed for maintainers or operators.

## Output Format

Use the user's language for the final review unless they request otherwise.

Start with findings, ordered by severity and confidence. Keep each finding concise:

```text
Must: <short title>
<file>:<line> - <scenario and impact>. <specific requested fix or verification>.
```

```text
Should: <short title>
<file>:<line> - <scenario and impact>. <specific improvement>.
```

After findings, add only the sections that are useful:
- `Open questions` for decisions or missing context that affect correctness.
- `Validation` for commands run and results, or validation not run.
- `Summary` as a brief secondary note. Do not let the summary hide findings.

If there are no Must or Should findings, say that clearly and mention any remaining validation gaps.

## Review Discipline

- Separate confirmed problems from questions.
- Do not label taste, formatting, or alternate-but-valid design as Must or Should.
- Do not ask for unrelated refactors.
- Do not require extra tests only for coverage theater; tie each requested test to a real risk.
- Prefer small, actionable fixes over vague advice.
- When evidence is incomplete, lower the severity or ask an open question instead of overstating the issue.
