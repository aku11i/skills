# Personal Agent Skills

This repository stores personal Agent Skills for repeated coding workflows.

## Install

Install all skills from this repository for Codex and Claude Code:

```bash
npx skills add aku11i/skills -g -a codex -a claude-code --skill '*' -y
```

Restart Codex or Claude Code after installation.

## Update

Update installed global skills:

```bash
npx skills update -g -y
```

Update a specific skill:

```bash
npx skills update gh-create-draft-pr -g -y
```

If new skills were added to this repository, run the install command again:

```bash
npx skills add aku11i/skills -g -a codex -a claude-code --skill '*' -y
```

Restart Codex or Claude Code after updating.

## Check Installed Skills

```bash
npx skills list -g
```

List skills available in this repository without installing:

```bash
npx skills add aku11i/skills --list
```
