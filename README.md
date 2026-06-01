# caes-skills

Shared caes-cru agent skills packaged as native Claude Code and Codex plugins.

## Install

Claude Code from a terminal:

```bash
claude plugin marketplace add ucdavis-caes/caes-skills
claude plugin install caes-skills@caes-skills
```

Claude Code from inside a session:

```text
/plugin marketplace add ucdavis-caes/caes-skills
/plugin install caes-skills@caes-skills
```

Codex from a terminal:

```bash
codex plugin marketplace add ucdavis-caes/caes-skills
```

Codex accepts GitHub repos, HTTPS/SSH Git URLs, and local marketplace root directories as marketplace sources. The current Codex CLI registers marketplaces with `codex plugin marketplace add`; install or enable the `caes-skills` plugin from Codex's plugin UI after the marketplace is added.

## Layout

```text
.agents/plugins/marketplace.json
.claude-plugin/marketplace.json
plugins/caes-skills/
  .claude-plugin/plugin.json
  .codex-plugin/plugin.json
  skills/
    devar/SKILL.md
    pixelify/SKILL.md
    pr-workflow/SKILL.md
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `devar` | DEVAR project scope and change discipline |
| `pixelify` | Show a team member's pixelated team image |
| `pr-workflow` | caes-cru's PR process |

## Adding a Skill

1. Create `plugins/caes-skills/skills/<name>/SKILL.md`:
   ```markdown
   ---
   name: your-skill-name
   description: One-line description of when to use this
   ---

   # Your Skill Title

   ...steps, rules, examples...
   ```

2. Commit and push

3. Bump the plugin version in `plugins/caes-skills/.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, and `plugins/caes-skills/.codex-plugin/plugin.json` when you want teammates to receive it as a release

## Skill Format

Skills use the same frontmatter format as Claude Code and Codex plugin skills:

- `name`: kebab-case identifier matching the directory name
- `description`: shown to the AI to decide when to load this skill
- Body: plain markdown — steps, tables, checklists, examples
