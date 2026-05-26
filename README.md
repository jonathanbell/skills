# skills

Personal collection of agent skills. Each skill follows the open Agent Skills
format: a folder containing a `SKILL.md` (markdown with YAML frontmatter) plus
optional `references/`, `scripts/`, and `evals/`. The format is portable across
any agent runtime that supports it.

## Skills

| Skill | Description |
|-------|-------------|
| [lighthouse-weather-report](./lighthouse-weather-report) | Fetches and decodes BC coast lightstation weather reports from Environment Canada into a clear, plain-English markdown briefing for 14 Vancouver Island stations. |

## Installing a skill

Copy the skill folder into the directory your agent runtime loads skills from.
The exact path depends on your runtime - consult its docs. As a concrete
example, one common location is:

```bash
cp -R <skill-name> ~/.claude/skills/
```

## Skill layout

```
<skill-name>/
  SKILL.md              # required: frontmatter + instructions
  references/           # optional: extended docs loaded as needed
  scripts/              # optional: executable helpers the skill calls
  evals/                # optional: test prompts and assertions
```

## License

[MIT](./LICENSE)
