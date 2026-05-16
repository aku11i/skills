---
name: gh-commit-and-push
description: Commit local repository changes and push them to the remote branch. Use when Codex is asked to commit and push changes, after inspecting recent commit message style and making a concise commit that matches the repository convention.
---

# Commit and Push

Use this skill to commit the current local changes and push them to the remote branch.

## Workflow

1. Inspect repository state.
   - Run `git status --short --branch`.
   - Review `git diff --stat` and `git diff` to understand the change.
   - Preserve unrelated user changes. Stage only files that belong in the requested commit.

2. Match the commit message style.
   - Inspect recent commits with `git log --oneline -5`.
   - If there are no prior commits, use a concise imperative message.
   - Do not add tool or agent prefixes such as `Codex:` or `AI:`.

3. Commit the change.
   - Stage the intended files.
   - Commit with a message that summarizes the actual change and follows the recent style.
   - Do not amend or rewrite existing commits unless the user explicitly asks.

4. Push the branch.
   - If the current branch has an upstream, run `git push`.
   - If no upstream exists, push with `git push -u origin <branch>`.
   - Report the commit hash and remote branch after the push succeeds.

## Decision Rules

- Ask before committing ambiguous unrelated changes.
- Ask before pushing if the remote, branch, or upstream target is unclear.
- Do not fabricate validation results. If tests or checks were not run, say so.

## Useful Commands

```bash
git status --short --branch
git diff --stat
git diff
git log --oneline -5
git branch --show-current
git push
git push -u origin <branch>
```
