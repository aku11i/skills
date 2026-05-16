---
name: gh-create-draft-pr
description: Create or update a GitHub draft pull request from the current repository state. Use when Codex is asked to create a draft PR, prepare a PR after local changes, push a branch, update an existing PR's title or body to match current diffs, or follow repository conventions for PR titles and descriptions.
---

# Create Draft PR

Use this skill to turn local repository changes into a GitHub draft pull request, or to reconcile an existing pull request with the current branch and diff.

## Workflow

1. Inspect repository state.
   - Run `git status --short --branch`.
   - Determine the current branch with `git branch --show-current`.
   - Determine the default branch from `gh repo view --json defaultBranchRef` when possible; fall back to `git remote show origin` or local branch names.
   - If the current branch is the default branch, create a new topic branch before committing or pushing. Use a concise branch name based on the change; avoid tool-specific names such as `codex/...`.

2. Understand the change before committing.
   - Review `git diff --stat`, `git diff`, and staged changes if present.
   - Preserve unrelated user changes. If the worktree contains changes outside the requested scope, do not include them unless the user explicitly wants them in the PR.
   - Check for relevant issue references, TODOs, design notes, or source files that explain why the change exists.
   - Run focused tests or validation commands appropriate to the change when feasible.

3. Commit as needed.
   - If there are uncommitted changes that belong in the PR, stage only those files.
   - Write a concise commit message following the repository's existing style, using recent commits as a guide.
   - Do not amend or rewrite existing commits unless the user explicitly asks.

4. Push the branch.
   - Push with upstream tracking when needed: `git push -u origin <branch>`.
   - If the branch already tracks a remote, use `git push`.

5. Check whether a PR already exists.
   - Use `gh pr view --json number,title,body,isDraft,state,headRefName,baseRefName,url` for the current branch.
   - If a PR exists, compare its title and body against the current branch diff and update it when they no longer match.
   - If the PR is already ready for review, do not convert it back to draft.

6. Learn repository PR conventions before writing title or body.
   - Inspect about five recently created PRs with `gh pr list --state all --limit 5 --json number,title,body,createdAt,url`.
   - Use their title style as the model for the new PR title.
   - Do not add prefixes that advertise a tool or agent, such as `Codex:`, `AI:`, or similar.
   - Check for PR templates in `.github/pull_request_template.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/PULL_REQUEST_TEMPLATE/`, `docs/`, or repository-specific template locations.

7. Draft the PR body.
   - If a template exists, fill it directly and preserve its headings, checklist style, and prompts.
   - If no template exists, mirror the structure and level of detail from recent PR bodies.
   - Summarize the user-visible or maintainer-relevant changes.
   - Include background, linked issues, related context, important source files, and external references when they help reviewers understand the change.
   - Include validation performed, such as tests run or commands checked. If validation was not run, state that clearly and briefly.

8. Create or update the PR.
   - Create new PRs as draft: `gh pr create --draft --title "<title>" --body-file <file>`.
   - Prefer `--body-file` over long inline shell strings to avoid quoting mistakes.
   - Update an existing PR with `gh pr edit --title "<title>" --body-file <file>` when the title or body is stale.
   - Report the PR URL and whether it was created, updated, or already current.

## Decision Rules

- Ask the user before including ambiguous unrelated changes in a commit or PR.
- Ask before pushing if the remote or branch target is unclear.
- Do not create a PR from the default branch; branch first.
- Do not downgrade a ready-for-review PR to draft.
- Do not use tool names, agent names, or automation prefixes in branch names, commit messages, PR titles, or PR body unless the repository already requires that exact convention.
- Do not fabricate issue links, test results, or external references. Include only sources actually inspected.

## Useful Commands

```bash
git status --short --branch
git branch --show-current
gh repo view --json defaultBranchRef
gh pr list --state all --limit 5 --json number,title,body,createdAt,url
gh pr view --json number,title,body,isDraft,state,headRefName,baseRefName,url
gh pr create --draft --title "<title>" --body-file <file>
gh pr edit --title "<title>" --body-file <file>
```
