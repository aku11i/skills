---
name: simplify
description: Simplify recently changed code by reviewing it for reuse, quality, efficiency, and project-instruction fit, then applying safe local improvements while preserving behavior.
---

# Simplify

Use this skill after implementation work when the user wants recently changed code cleaned up, simplified, or made easier to maintain without changing behavior.

## Target Selection

1. Inspect repository state first.
   - Run `git status --short --branch`.
   - If the worktree has uncommitted changes, simplify those changes by default.
   - Include staged and unstaged changes. Check untracked files when they are relevant to the requested work.

2. If there are no uncommitted changes, inspect the current branch changes.
   - Prefer `gh pr view --json number,title,body,baseRefName,headRefName,url` and `gh pr diff` when a pull request exists.
   - Otherwise compare the current branch with the likely base branch.
   - If the target is still unclear, ask which files, commit range, or PR to simplify.

3. Respect any focus text from the user.
   - Treat focus text such as memory use, duplication, readability, startup time, or a specific subsystem as the highest-priority lens.
   - Do not ignore obvious nearby simplifications that are low risk and inside the same changed surface.

## Workflow

1. Understand intent and constraints.
   - Read the user request, changed files, surrounding code, tests, and project instructions such as `AGENTS.md`.
   - Identify the behavior that must be preserved before editing.
   - Avoid large rewrites, architecture changes, dependency additions, or public API changes unless they are clearly necessary and within scope.

2. Review the changed code through these lenses.
   - Reuse: duplicated logic, repeated literals, missed local helpers, unnecessary custom code, and opportunities to align with existing project patterns.
   - Quality: naming, control flow, nesting, branching, error handling, dead code, unnecessary indirection, misleading abstractions, and code that is harder to read than the problem requires.
   - Efficiency: avoidable repeated work, excessive allocation, inefficient loops or queries, unnecessary network or filesystem calls, slow startup paths, and resource use that matters for realistic inputs.
   - Project fit: repository conventions, formatting, instructions, test style, generated-file rules, and compatibility with existing APIs or data formats.

3. Use independent review passes when allowed.
   - If the user explicitly requested subagents, parallel review, or delegated work, ask separate agents to inspect reuse, quality, and efficiency.
   - Otherwise perform the same passes yourself.
   - Reconcile conflicts by preserving behavior first, then choosing the simplest change that matches local patterns.

4. Apply safe simplifications.
   - Keep edits tightly scoped to the selected target.
   - Prefer deleting unnecessary code over adding new abstractions.
   - Extract helpers only when duplication is real and the helper name makes the call sites clearer.
   - Flatten control flow when it improves readability, but keep domain checks explicit.
   - Replace clever expressions with clear code when that makes maintenance easier.
   - Preserve unrelated user changes; do not revert or reformat files outside the simplification target.

5. Validate behavior.
   - Run focused tests, type checks, linters, formatters, or build commands that are cheap and relevant.
   - If there are no suitable commands, perform a careful manual diff review and state that validation was manual.
   - If a command is unavailable, slow, risky, or blocked by credentials, state the gap.

6. Report the result.
   - Summarize what was simplified and why.
   - List validation commands and outcomes.
   - Mention any intentionally skipped opportunities that would require a broader refactor or product decision.

## Simplification Rules

- Behavior preservation is mandatory. Do not change inputs, outputs, persistence, permissions, errors, timing-sensitive behavior, or user-visible text unless explicitly requested.
- Do not invent abstractions for single-use code.
- Do not chase style-only nits. Only edit when the result is clearly simpler, safer, faster, or more consistent.
- Do not remove tests unless they are redundant after confirming equivalent coverage remains.
- Do not hide important edge cases behind overly generic helpers.
- Prefer local consistency over generic best practices when the two differ.

## Useful Commands

```bash
git status --short --branch
git diff --stat
git diff
git diff --staged
git log --oneline -5
gh pr view --json number,title,body,baseRefName,headRefName,url
gh pr diff
```
