# Kibo Agent catalog

Skills, slash commands, runbooks, MCP servers and hooks for [Kibo Agent](https://github.com/gribanoveu/kibo-agent),
installed from Settings → Catalog. The app reads this repository over plain HTTP — `index.json`
first, then the files of what you install — so it needs neither `git` nor a token.

Base URL for the app's settings:

```
https://raw.githubusercontent.com/gribanoveu/kibo-agent-catalog/main/
```

## Layout

```
index.json                      what the app lists
skills/<id>/SKILL.md            a skill: a folder, SKILL.md plus any files beside it
commands/<id>.md                a slash command
runbooks/<role>/<id>.md         a runbook for a role (kubernetes)
mcp/<id>.json                   one MCP server entry, without its name
hooks/<id>.json                 a {"hooks": {...}} fragment in hooks.json format
```

## index.json

```json
{
  "version": 1,
  "items": [
    { "id": "review", "kind": "skill", "name": "Code review", "description": "…",
      "rev": "1", "path": "skills/review", "files": ["SKILL.md", "checklist.md"] }
  ]
}
```

- `id` — `[a-z0-9-]`; the name the item is installed under (skill folder, command or runbook
  file, key in `mcpServers`). A skill's `id` must equal the `name` in its `SKILL.md`.
- `kind` — `skill`, `command`, `runbook`, `mcp` or `hooks`. A runbook also has `role`.
- `rev` — any string; change it whenever the item's files change, or installed copies will not
  be offered the update.
- `path` — a folder for a skill (with `files` relative to it), the single file otherwise.
  Relative, no `..`.

## Secrets

Never commit a secret. Write a placeholder `${NAME}` in an MCP entry's `env` or `headers`; the
app asks for its value when the server is installed and writes it only to the user's `mcp.json`.

## Adding an item

1. Add the files under the folder of its kind.
2. Add an entry to `index.json`.
3. Open a pull request. MCP servers and hooks run commands on the user's machine — say what the
   command does and why it is safe.
