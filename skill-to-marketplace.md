# From Skill to Marketplace: A Step-by-Step Guide

This guide walks you through authoring a Claude Code skill in this repo, wrapping it in a plugin, publishing a marketplace from the repo on GitHub, and letting others install it.

**Mental model:** a **skill** is one focused capability (a `SKILL.md` file). A **plugin** is a versioned bundle that can contain skills, agents, hooks, MCP servers, etc. A **marketplace** is a repo that lists one or more plugins so users can `/plugin install` them.

So the chain is: `SKILL.md` → plugin folder → marketplace.json → GitHub → users.

---

## Step 1 — Decide what the skill does

Pick one focused capability and a kebab-case name (e.g. `pr-summarizer`, `stijns-greeter`). The skill's `description` is what Claude reads to decide when to auto-invoke it, so be concrete about *when* to use it, not just *what* it does.

Rules of thumb:
- **Name:** lowercase, hyphens, ≤ 64 chars.
- **Description:** combined `description` + `when_to_use` caps at **1,536 characters**. Front-load the trigger.
- One skill = one job. If it sprawls, split it.

---

## Step 2 — Lay out the repo

From the repo root (`/workspaces/stijns-super-skilzz`), create this structure:

```
stijns-super-skilzz/
├── .claude-plugin/
│   └── marketplace.json          # the marketplace manifest
└── plugins/
    └── <plugin-name>/
        ├── .claude-plugin/
        │   └── plugin.json       # the plugin manifest
        └── skills/
            └── <skill-name>/
                └── SKILL.md      # the skill itself
```

```bash
mkdir -p .claude-plugin
mkdir -p plugins/<plugin-name>/.claude-plugin
mkdir -p plugins/<plugin-name>/skills/<skill-name>
```

> ⚠️ Only `plugin.json` goes inside `.claude-plugin/`. Put `skills/`, `agents/`, `hooks/` at the plugin root — not nested under `.claude-plugin/`.

---

## Step 3 — Write `SKILL.md`

`plugins/<plugin-name>/skills/<skill-name>/SKILL.md`:

```markdown
---
name: stijns-greeter
description: Greets the user in Stijn's particular style. Use when the user asks for a greeting, salutation, or warm welcome message.
---

# Stijns Greeter

When invoked, greet the user warmly in Stijn's voice. Ask their name if it isn't known, then produce a one-line greeting.

## Steps

1. Determine the user's name (from memory or by asking).
2. Respond with: `Hey <name>, Stijn sends his regards!`
```

Frontmatter fields you'll most likely want:

| Field | Purpose |
|---|---|
| `name` | Display name (kebab-case). Defaults to folder name. |
| `description` | Required in practice — drives auto-invocation. |
| `when_to_use` | Extra trigger context. Counts toward the 1,536-char cap. |
| `allowed-tools` | Pre-approve tools, e.g. `Bash(git status:*) Read`. |
| `disable-model-invocation` | `true` = only runs when user types `/skill-name`. |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]`. |

Reference: https://code.claude.com/docs/en/skills.md

---

## Step 4 — Write `plugin.json`

`plugins/<plugin-name>/.claude-plugin/plugin.json`:

```json
{
  "name": "stijns-super-plugin",
  "description": "Stijn's personal bag of tricks.",
  "version": "0.1.0",
  "author": { "name": "Stijn" }
}
```

- `name` is the namespace users see: skills become `/stijns-super-plugin:stijns-greeter`.
- Omit `version` to let the commit SHA act as the version; set one explicitly if you want controlled updates.

Reference: https://code.claude.com/docs/en/plugins.md

---

## Step 5 — Write `marketplace.json`

`.claude-plugin/marketplace.json` at the repo root:

```json
{
  "name": "stijns-super-skilzz",
  "owner": {
    "name": "Stijn",
    "email": "sven.siertsema@gmail.com"
  },
  "plugins": [
    {
      "name": "stijns-super-plugin",
      "source": "./plugins/stijns-super-plugin",
      "description": "Stijn's personal bag of tricks."
    }
  ]
}
```

- `source: "./plugins/..."` is a relative path from the marketplace file. Don't use `../`.
- The marketplace `name` is what users will suffix on install: `@stijns-super-skilzz`.

Reference: https://code.claude.com/docs/en/plugin-marketplaces.md

---

## Step 6 — Test locally before pushing

From the repo root:

```
/plugin marketplace add ./
/plugin install stijns-super-plugin@stijns-super-skilzz
/reload-plugins
```

Then try it:

```
/stijns-super-plugin:stijns-greeter
```

If you edit `SKILL.md`, most changes are live — otherwise run `/reload-plugins`. If the skill doesn't show up, check:
- `plugin.json` and `marketplace.json` are valid JSON.
- `SKILL.md` frontmatter is valid YAML (quote anything with colons or special chars).
- `disable-model-invocation` isn't accidentally `true` if you expect auto-invocation.

---

## Step 7 — Commit and push to GitHub

```bash
git add .claude-plugin plugins skill-to-marketplace.md
git commit -m "Add stijns-super-plugin marketplace"
git push origin main
```

Make sure the repo is public (or collaborators have access) so others can install.

---

## Step 8 — Share the install command

Anyone can now install with:

```
/plugin marketplace add <github-user>/stijns-super-skilzz
/plugin install stijns-super-plugin@stijns-super-skilzz
/reload-plugins
```

---

## Step 9 — Updating and versioning

- Bump `version` in `plugin.json` when you ship breaking or notable changes; users only pull updates on a bump (if `version` is set).
- If you omit `version`, every new commit on `main` becomes the latest version.
- Users update via `/plugin update` and remove via `/plugin uninstall`.

---

## Common gotchas

| Symptom | Fix |
|---|---|
| Skill doesn't auto-invoke | `description` is vague or too long — rewrite with explicit "Use when…" triggers. |
| `/plugin install` fails with path error | `source` in `marketplace.json` uses `../` or absolute path — keep it `./plugins/<name>`. |
| Edits don't show up | Plugins are cached under `~/.claude/plugins/cache/`; run `/reload-plugins`. |
| Frontmatter silently ignored | YAML syntax error — quote strings, check indentation. |
| Skill name rejected | Must be kebab-case, no spaces, no uppercase. |
| Nested `skills/` under `.claude-plugin/` | Move `skills/` to the plugin root; only `plugin.json` lives in `.claude-plugin/`. |

---

## Reference links

- Skills: https://code.claude.com/docs/en/skills.md
- Creating plugins: https://code.claude.com/docs/en/plugins.md
- Plugin marketplaces: https://code.claude.com/docs/en/plugin-marketplaces.md
- Discovering plugins: https://code.claude.com/docs/en/discover-plugins.md
- Plugins reference: https://code.claude.com/docs/en/plugins-reference.md
