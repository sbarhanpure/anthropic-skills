# anthropic-skills

Sandeep's personal skill catalog. Portable expertise for AI-assisted engineering work. SKILL.md format per the Anthropic Agent Skills specification (`agentskills.io`).

## Skills

| Skill | Purpose |
|---|---|
| [`pr-reviewer`](./pr-reviewer/) | Structured PR review with blocking/nit distinction and termination rules |
| [`prd-validator`](./prd-validator/) | Validate a PRD against engineering-ready standards |
| [`adr-writer`](./adr-writer/) | Produce Nygard-format Architecture Decision Records |
| [`tier-classifier`](./tier-classifier/) | Classify tickets as S / M / L with explicit rubric |
| [`tdd-implementer`](./tdd-implementer/) | Test-first implementation discipline |

## Format

Each skill is a folder with:
- `SKILL.md` — YAML frontmatter (name, description) + markdown body with Examples and Guidelines sections
- `references/` — additional detail loaded only when needed (progressive disclosure)
- `scripts/` (optional) — deterministic checks or generators

## Use in Claude Code

Symlink to project's `.claude/skills/` or user's `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
ln -s $(pwd)/pr-reviewer ~/.claude/skills/
ln -s $(pwd)/prd-validator ~/.claude/skills/
ln -s $(pwd)/adr-writer ~/.claude/skills/
ln -s $(pwd)/tier-classifier ~/.claude/skills/
ln -s $(pwd)/tdd-implementer ~/.claude/skills/
```

Claude auto-discovers skills based on their description field. To trigger explicitly: *"Use the prd-validator skill on this draft."*

## Use in Claude.ai

Upload skills via the skills UI on paid plans. See [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude).

## Use in the API

Per the [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide).

## Source

Format and conventions follow:
- [`anthropics/skills`](https://github.com/anthropics/skills) — official Anthropic catalog
- [`anthropics/skills/skills/skill-creator`](https://github.com/anthropics/skills/tree/main/skills/skill-creator) — meta-skill for creating skills
- [agentskills.io](https://agentskills.io) — open specification

## License

Apache 2.0 unless otherwise noted in a specific skill.
