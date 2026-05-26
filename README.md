# skills

Personal collection of [Claude Code](https://docs.claude.com/en/docs/claude-code/overview)
skills. Each skill lives in its own top-level folder with a `SKILL.md` and
optional `references/`, `scripts/`, and `evals/`.

## Skills

| Skill | Description |
|-------|-------------|
| [lighthouse-weather-report](./lighthouse-weather-report) | Fetches and decodes BC coast lightstation weather reports from Environment Canada into a clear, plain-English markdown briefing for 14 Vancouver Island stations. |

## Installing a skill

Copy the skill folder into your Claude Code skills directory:

```bash
cp -R <skill-name> ~/.claude/skills/
```

Claude Code picks it up automatically on the next session. Verify with:

```bash
ls ~/.claude/skills/
```

## Skill layout

```
<skill-name>/
  SKILL.md              # required: frontmatter + instructions
  references/           # optional: extended docs loaded as needed
  scripts/              # optional: executable helpers the skill calls
  evals/                # optional: test prompts for the skill-creator loop
```

See the [Anthropic skills documentation](https://docs.claude.com/en/docs/claude-code/skills)
for the full format.

## License

[MIT](./LICENSE)
