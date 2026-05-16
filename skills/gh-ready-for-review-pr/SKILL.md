---
name: gh-ready-for-review-pr
description: Mark an existing GitHub draft pull request as ready for review. Use when Codex is asked to make a PR ready for review, convert a draft PR to ready, or publish a draft PR for reviewer attention after confirming the PR contents are acceptable.
---

# Ready for Review PR

Use this skill to convert an existing draft pull request to ready for review.

## Workflow

1. Identify the current pull request.
   - Run `gh pr view --json number,title,body,isDraft,state,url`.
   - If no PR exists for the current branch, stop and report that there is no PR to update.

2. Confirm the PR can be made ready.
   - Review the PR title and body for obvious mismatch with the current branch diff.
   - Check `git status --short --branch` and make sure there are no uncommitted changes that should be included first.
   - If the PR is already ready for review, report that no change is needed.
   - If the PR content appears incomplete or stale, update the PR or ask the user before marking it ready.

3. Mark the PR as ready for review.
   - Run `gh pr ready`.
   - Report the PR URL and the resulting state.

## Useful Commands

```bash
git status --short --branch
gh pr view --json number,title,body,isDraft,state,url
gh pr ready
```
