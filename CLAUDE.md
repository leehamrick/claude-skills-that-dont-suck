# Claude Skills Workshop

This project is a development workspace for Claude Code skills. Skills here are authored and tested, then installed separately for use.

## Project structure

- `skills/` — each subfolder is one skill, containing a `SKILL.md`
- `dist/` — packaged `.skill` files ready for distribution (gitignored, built on demand)

## Conventions

- One folder per skill under `skills/`, named with the skill's trigger word (e.g., `de-fluff`, `handoff`)
- Each skill folder contains exactly one `SKILL.md` at the root — no nesting
- Skill names use lowercase kebab-case
- When editing a skill, validate that the description frontmatter contains accurate trigger phrases
- When creating a new skill, check for dependencies on other skills in this repo and note them in the skill's SKILL.md

## Packaging

To package a skill for distribution:
```bash
tar czf dist/<skill-name>.skill -C skills/<skill-name> .
```

## Install instructions (for end users)

Windows (PowerShell):
```powershell
mkdir "$env:USERPROFILE\.claude\skills\<skill-name>" -Force
tar xzf <skill-name>.skill -C "$env:USERPROFILE\.claude\skills\<skill-name>"
```

macOS / Linux:
```bash
mkdir -p ~/.claude/skills/<skill-name>
tar xzf <skill-name>.skill -C ~/.claude/skills/<skill-name>
```

Or just copy the `SKILL.md` directly into `~/.claude/skills/<skill-name>/`.
