---
name: gh-address-comments
description: Address actionable GitHub pull request review comments. Use when Codex is asked to inspect unresolved, unreplied, or unaddressed PR review feedback, decide whether each comment is a Must or Should issue, implement accepted fixes, commit and push them, then reply to and resolve review threads.
---

# Address PR Review Comments

Use this skill to work through GitHub pull request review comments from discovery to resolution.

## Workflow

1. Identify the pull request.
   - If the user provides a PR URL, repository, or number, use that directly.
   - Otherwise, run `gh pr view --json number,title,body,baseRefName,headRefName,url` from the current branch.
   - If the PR cannot be resolved, ask for the missing repository or PR identifier.

2. Understand the PR before judging comments.
   - Read the PR title, body, linked issues, recent commits, and changed files.
   - Inspect the diff with `gh pr diff` or an equivalent base comparison.
   - Identify the intended scope, implementation background, and constraints before deciding whether feedback applies.

3. Fetch thread-aware review comments.
   - Prefer GitHub review threads over flat comment lists because resolution state and reply order matter.
   - Fetch unresolved review threads with `gh api graphql`, including `id`, `isResolved`, `isOutdated`, `path`, `line`, `originalLine`, `diffSide`, and each comment's `id`, `author`, `body`, `createdAt`, and `url`.
   - Ignore resolved threads by default. Treat outdated threads as lower priority unless the concern still applies to the current diff.

4. Select comments that still need action.
   - Focus on unresolved threads where the latest substantive reviewer comment has no later author response.
   - Skip comments already addressed by the current diff, duplicate comments covered by another thread, and comments that only require no-op acknowledgement.
   - For each remaining thread, decide whether it is `Must`, `Should`, or `Nit`.
   - `Must`: correctness, security, data integrity, compatibility, failing validation, or missing required behavior.
   - `Should`: meaningful maintainability, test, observability, performance, or documentation improvement that fits this PR.
   - `Nit`: preference, style, optional cleanup, broad refactor, or advice outside the PR scope. Do not implement nits unless the user explicitly asks.

5. Decide and document the response plan.
   - Address `Must` and relevant `Should` comments.
   - Do not address comments that conflict with the PR goal, are factually incorrect, duplicate already-handled feedback, or are only nits.
   - If a comment is ambiguous or would materially change scope, ask the user before editing.
   - Keep a concise mapping of thread IDs to the intended action or non-action reason.

6. Implement accepted fixes locally.
   - Keep changes scoped to the review feedback and current PR.
   - Preserve unrelated user changes; stage only files changed for the selected review comments.
   - Run focused tests, linters, type checks, or other relevant validation when feasible.

7. Commit and push.
   - Review `git status --short --branch`, `git diff --stat`, and `git diff`.
   - Inspect recent commit messages with `git log --oneline -5`.
   - Commit only the review-response changes with a concise message matching repository style.
   - Push to the PR branch with `git push` or `git push -u origin <branch>` when no upstream exists.

8. Reply to every selected thread and resolve it.
   - For implemented fixes, reply with what changed and any validation that supports it.
   - For comments not implemented, reply with the concrete reason, such as out of scope, already handled, not applicable to the PR goal, or nit-level feedback.
   - After replying, resolve the review thread.
   - Do not resolve a thread until the reply has been posted.

9. Report the outcome.
   - List addressed threads, intentionally not-addressed threads, commit hash, push target, and validation performed.
   - If validation was not run, state that clearly.
   - Mention any remaining unresolved or ambiguous review threads.

## GitHub CLI Notes

Use `gh api graphql` for review thread state and mutations. The important mutations are:

- `addPullRequestReviewThreadReply` to reply to a thread.
- `resolveReviewThread` to mark the thread resolved.

Use `--field` or a variables file for GraphQL inputs rather than embedding long JSON in shell strings.

## Safety Rules

- Do not treat every review comment as automatically valid. Judge it against the PR purpose, implementation background, and Must/Should/Nit severity.
- Do not implement nits, broad rewrites, or unrelated cleanup by default.
- Do not push, reply, or resolve threads until local fixes and validation are complete.
- Do not fabricate validation results, reviewer intent, or issue context.
- Ask the user before handling comments that require product decisions, large scope changes, or conflicting reviewer guidance.

## Useful Commands

```bash
gh pr view --json number,title,body,baseRefName,headRefName,url
gh pr diff
git status --short --branch
git diff --stat
git diff
git log --oneline -5
git push
```
