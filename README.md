# Host skills

Claude Code skills shared by the team. DevFlow's **Skills Marketplace** module reads this repository and installs any skill from it with one click, globally (`~/.claude/skills/<name>`) or into one repository (`<repo>/.claude/skills/<name>`).

## Layout

```
skills.json            the catalog: one entry per skill
skills/<name>/SKILL.md the skill, plus any files it references
```

`skills.json`:

```json
{
  "name": "Host skills",
  "skills": [
    { "name": "unslop", "description": "Cut AI tells from any writing", "version": "1.0.0", "tags": ["writing"], "author": "Ricardo Silva" }
  ]
}
```

- `name` is the folder under `skills/` and the name Claude resolves. Letters, digits, dots, hyphens and underscores only.
- `path` is optional and defaults to `skills/<name>`. The folder must hold a `SKILL.md`.
- `version` is free text. DevFlow shows **Update available** when it differs from the installed copy, so bump it when you change a skill.
- `tags` and `author` feed the card and the search.

## Adding a skill

1. Copy the skill folder into `skills/<name>/`. Keep the `SKILL.md` frontmatter (`name`, `description`); DevFlow falls back to it when the catalog entry has no description.
2. Add an entry to `skills.json`.
3. Commit and push. Everyone sees it on their next **Refresh**.

## Using it in DevFlow

Settings → Claude Skills → **Marketplace sources** → add `https://github.com/ricardodsilva/marketplace.git` → Save → sidebar **Skills Marketplace**.
