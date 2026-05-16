# Repository Guidelines

This repository stores personal Agent Skills for repeated use across coding workflows.

## Purpose

- Treat this repository as a personal skill catalog, not an application codebase.
- Keep skills practical, reusable, and focused on workflows the owner actually uses.
- Prefer clear procedural guidance over broad documentation.

## Skill Layout

- Place installable skills under `skills/<skill-name>/`.
- Each skill must include `SKILL.md`.
- Each skill should include `agents/openai.yaml` when useful for Codex UI metadata.
- Use lowercase, hyphenated skill names such as `gh-create-draft-pr`.
- Do not place skill directories at the repository root.

## Language

- Write repository-managed source, documentation, and generated artifacts in English.
- User-facing conversation may be in Japanese, but committed files should remain English.

## Skill Authoring

- Keep `SKILL.md` concise and action-oriented.
- Include only information another agent needs to perform the workflow reliably.
- Avoid extra README, changelog, or guide files unless they are directly useful to the skill.
- Do not include tool or agent branding in generated workflow outputs unless the target repository explicitly requires it.
- Prefer commands and decision rules that preserve unrelated user changes.

## Validation

- Check that each `SKILL.md` has valid YAML frontmatter with `name` and `description`.
- Check that new or edited files do not contain unintended Japanese text.
- When scripts are added to a skill, run representative tests before committing.
- If a validation tool is unavailable in the environment, document what was checked manually.

## Git Workflow

- Inspect recent commits before choosing commit messages.
- Keep commits scoped to one logical change.
- Push completed changes to the tracked remote branch when requested.
