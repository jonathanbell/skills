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

The exact install method depends on the runtime. Common paths are below.

### Claude apps (desktop, web, and mobile)

Skills are uploaded to your Claude.ai account and become available everywhere
you sign in - desktop app, web, and mobile.

1. **Enable the prerequisite once.** In Claude, open
   `Settings → Capabilities` and turn on **Code Execution and File Creation**.
   Skills will not run without this.
2. **Package the skill folder as a ZIP.** From the repo root:
   ```bash
   zip -r <skill-name>.zip <skill-name>
   ```
   The ZIP must contain the skill folder with `SKILL.md` inside.
3. **Upload it.** In Claude, open `Settings → Customize → Skills`, click the
   `+` button, choose **Upload**, and select the ZIP. The skill appears in
   your list immediately and stays private to your account.
4. **Use it on mobile.** Sign in to the Claude mobile app with the same
   account - your skills sync automatically.

### Claude Code (CLI)

Copy the skill folder into your skills directory:

```bash
cp -R <skill-name> ~/.claude/skills/
```

Use `~/.claude/skills/` for personal skills (available everywhere) or
`.claude/skills/` inside a project for project-scoped skills.

### Other runtimes

Place the skill folder wherever the runtime loads skills from - consult its
docs for the path.

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
