# What We're Building

A **Claude Code plugin marketplace** — a set of reusable slash commands that standardize common dev workflows across projects and teams, distributed via Claude Code's native plugin system.

## Repo Structure

This repo serves as both a **marketplace** (the index) and the **plugin** (the actual tool). Here's the layout:

```
xClaude-dev-tools/
├── .claude-plugin/
│   └── marketplace.json                  ← Marketplace index (lists available plugins)
├── plugins/
│   └── xclaude-dev-tools/                ← The actual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json               ← Plugin manifest (name, version, description)
│       └── commands/                     ← Slash commands (markdown files)
│           ├── generate-commit.md
│           ├── review.md
│           ├── pr-review.md
│           ├── pull-request.md
│           ├── sync-pr.md
│           ├── migration-review.md
│           └── test-for.md
├── README.md
└── WHAT-WE-ARE-BUILDING.md
```

Two key pieces:

1. **`.claude-plugin/marketplace.json`** at the repo root — makes this repo a marketplace. It's an index that tells Claude Code "here are the plugins available, and here's where to find them."
2. **`plugins/xclaude-dev-tools/`** — the actual plugin folder. Contains its own `.claude-plugin/plugin.json` manifest and the `commands/` directory.

Later, when we want to separate concerns, the marketplace can live in its own repo and point to plugin repos via GitHub URLs instead of relative paths.

---

## How the Pieces Fit Together

### The Marketplace (`marketplace.json`)

A marketplace is just a JSON file that lists plugins and where to find them:

```json
{
  "name": "keyvalue-tools",
  "owner": { "name": "KeyValue" },
  "plugins": [
    {
      "name": "xclaude-dev-tools",
      "source": "./plugins/xclaude-dev-tools",
      "description": "Slash commands for dev workflows"
    }
  ]
}
```

- **`name`**: The marketplace identifier. Users reference this when installing (`xclaude-dev-tools@keyvalue-tools`).
- **`owner`**: Who maintains the marketplace.
- **`plugins`**: Array of plugin entries. Each has a `name` and a `source` (where to find it).
- **`source`**: Can be a relative path (like `./plugins/xclaude-dev-tools`), a GitHub repo (`{"source": "github", "repo": "owner/repo"}`), a git URL, an npm package, or even a pip package.

### The Plugin (`plugin.json`)

Each plugin has its own manifest:

```json
{
  "name": "xclaude-dev-tools",
  "description": "Claude Code slash commands for development workflows",
  "version": "1.0.0",
  "author": { "name": "KeyValue" },
  "license": "MIT"
}
```

- **`name`**: The plugin identifier. Becomes the namespace for all commands (e.g., `/xclaude-dev-tools:review`).
- **`version`**: Semantic versioning. Claude Code uses this to detect updates. **You must bump this when you push changes**, otherwise users won't see updates due to caching.
- The rest is metadata.

### Commands (`commands/*.md`)

Each `.md` file becomes a slash command. The filename (minus `.md`) is the command name, prefixed with the plugin namespace:

| File                   | Slash Command                        | What it does                                              |
|------------------------|--------------------------------------|-----------------------------------------------------------|
| `generate-commit.md`  | `/xclaude-dev-tools:generate-commit` | Interactively stage files and create conventional commits |
| `review.md`           | `/xclaude-dev-tools:review`          | Review git diff for correctness, security, architecture   |
| `pr-review.md`        | `/xclaude-dev-tools:pr-review`       | Review a GitHub PR with the same review criteria          |
| `pull-request.md`     | `/xclaude-dev-tools:pull-request`    | Create or update a GitHub PR with structured description  |
| `sync-pr.md`          | `/xclaude-dev-tools:sync-pr`         | Refresh PR description after new commits                  |
| `migration-review.md` | `/xclaude-dev-tools:migration-review`| Review database migrations for safety                     |
| `test-for.md`         | `/xclaude-dev-tools:test-for`        | Generate unit tests for a class                           |

Each markdown file is a **prompt** — instructions that tell Claude what to do when the command is invoked. For example, `generate-commit.md` tells Claude to: analyze the diff → let user pick files → draft a conventional commit message → confirm → commit → optionally push.

The text a user types after the command name is available as `$ARGUMENTS` inside the prompt. For example:
```
/xclaude-dev-tools:test-for CustomerService
```
Here, `CustomerService` becomes `$ARGUMENTS` in the `test-for.md` prompt.

---

## How to Use It

### For development/testing (local)

Load the plugin directly without installing:

```bash
claude --plugin-dir /path/to/xClaude-dev-tools/plugins/xclaude-dev-tools
```

Or test the full marketplace flow locally:
```
/plugin marketplace add /path/to/xClaude-dev-tools
/plugin install xclaude-dev-tools@keyvalue-tools
```

### For teammates / anyone else

Once this repo is pushed to GitHub (e.g., `KeyValue/xClaude-dev-tools`), anyone can install it with two commands inside Claude Code:

```
/plugin marketplace add KeyValue/xClaude-dev-tools
/plugin install xclaude-dev-tools@keyvalue-tools
```

**Step 1** adds the marketplace (the index). **Step 2** installs a specific plugin from that marketplace. After this, all 7 commands are available in every Claude Code session.

### For a whole team (auto-setup)

To make the marketplace available automatically when teammates clone a project, add this to the project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "keyvalue-tools": {
      "source": {
        "source": "github",
        "repo": "KeyValue/xClaude-dev-tools"
      }
    }
  },
  "enabledPlugins": {
    "xclaude-dev-tools@keyvalue-tools": true
  }
}
```

When a teammate opens the project in Claude Code, they'll be prompted to install the marketplace and plugin automatically.

---

## Updating the Plugin

### As a developer (pushing changes)

1. Edit the command `.md` files in `plugins/xclaude-dev-tools/commands/`
2. **Bump the version** in `plugins/xclaude-dev-tools/.claude-plugin/plugin.json` (e.g., `1.0.0` → `1.1.0`)
3. Commit and push to GitHub

The version bump is critical — Claude Code caches plugins locally and uses the version to detect changes.

### As a user (pulling updates)

```
/plugin marketplace update keyvalue-tools
/plugin update xclaude-dev-tools@keyvalue-tools
```

Or Claude Code may detect the new version and prompt automatically.

### During development (hot reload)

If testing with `--plugin-dir`, just run inside Claude Code:
```
/reload-plugins
```

No restart needed.

---

## What the Plugin System Supports (Beyond Commands)

We're currently using **commands only**, but plugins can include much more:

| Component       | Directory/File  | What it does                                                  |
|-----------------|-----------------|---------------------------------------------------------------|
| **Commands**    | `commands/`     | Slash commands invoked by the user (what we have now)         |
| **Skills**      | `skills/`       | Auto-invoked by Claude based on task context (no slash needed)|
| **Agents**      | `agents/`       | Specialized sub-agents with own system prompts and tools      |
| **Hooks**       | `hooks/`        | Event handlers (e.g., auto-lint after every file write)       |
| **MCP servers** | `.mcp.json`     | Connect Claude to external tools/APIs                         |
| **LSP servers** | `.lsp.json`     | Give Claude real-time code intelligence (go-to-def, etc.)     |
| **Settings**    | `settings.json` | Default config applied when plugin is enabled                 |

### Skills vs Commands

- **Commands** (`commands/`): User types `/xclaude-dev-tools:review` to invoke. Explicit.
- **Skills** (`skills/`): Claude automatically uses them based on the task. For example, a "code-review" skill could be auto-triggered whenever Claude sees a PR. Each skill has a `SKILL.md` with a `description` field that Claude uses to decide when to invoke it.

### Hooks

Event-driven scripts that run automatically:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint:fix" }]
      }
    ]
  }
}
```

Available events: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `SessionEnd`, `Stop`, and more.

### Agents

Sub-agents with specialized roles:

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
---

You are a security expert. When reviewing code, focus on...
```

These appear in `/agents` and Claude can invoke them automatically for relevant tasks.

### MCP Servers

Connect Claude to external tools via the Model Context Protocol:

```json
{
  "mcpServers": {
    "my-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

`${CLAUDE_PLUGIN_ROOT}` is a special variable that resolves to the plugin's install location.

We can add any of these to our plugin as needed.

---

## How This Differs From What We Had Before

Previously, we built an **npm package** that:
1. Ships a Node.js CLI (`bin/cli.js`)
2. On `npm install`, runs a `postinstall` script
3. The script **copies** `.md` files into `.claude/commands/` in the project

| Aspect              | Old (npm package)                      | New (native plugin)                    |
|---------------------|----------------------------------------|----------------------------------------|
| Install mechanism   | `npm install` + file copy              | `claude plugin install` (native)       |
| Distribution        | npm registry                           | Git repo marketplace                   |
| Dependencies        | Requires Node.js                       | Zero dependencies                      |
| Capabilities        | Commands only                          | Commands + Skills + Agents + Hooks + MCP + LSP |
| Updates             | `npm update` + re-copy                 | `/plugin update` (versioned, cached)   |
| Uninstall           | Custom CLI to clean up copied files    | `/plugin uninstall` (clean)            |
| Team sharing        | npm package in `package.json`          | `--scope project` or `extraKnownMarketplaces` |
| Sync issues         | Copied files can drift from source     | Plugin is cached as a unit, always in sync |

The official plugin system eliminates all the npm machinery. The plugin folder **is** the distribution unit.

---

## Distribution Options (Future)

### Option 1: Same repo (current — for testing)

Marketplace and plugin live in the same repo. Simple but mixes concerns.

### Option 2: Separate repos (recommended for production)

- **Marketplace repo** (`KeyValue/claude-plugins-marketplace`): Contains only `marketplace.json`, points to plugin repos via GitHub URLs
- **Plugin repo(s)** (`KeyValue/xClaude-dev-tools`): Contains only the plugin

The marketplace becomes an index that can list multiple plugins from different repos:

```json
{
  "name": "keyvalue-tools",
  "owner": { "name": "KeyValue" },
  "plugins": [
    {
      "name": "xclaude-dev-tools",
      "source": { "source": "github", "repo": "KeyValue/xClaude-dev-tools" }
    },
    {
      "name": "xclaude-testing-tools",
      "source": { "source": "github", "repo": "KeyValue/xClaude-testing-tools" }
    }
  ]
}
```

### Option 3: Official Anthropic marketplace

Submit at [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit) for public discovery.

---

## Quick Reference

```bash
# Test plugin locally (no install)
claude --plugin-dir ./plugins/xclaude-dev-tools

# Test full marketplace flow locally
# (inside Claude Code)
/plugin marketplace add /path/to/xClaude-dev-tools
/plugin install xclaude-dev-tools@keyvalue-tools

# Install from GitHub (what teammates run)
/plugin marketplace add KeyValue/xClaude-dev-tools
/plugin install xclaude-dev-tools@keyvalue-tools

# Hot reload after editing commands
/reload-plugins

# Debug plugin loading
claude --debug --plugin-dir ./plugins/xclaude-dev-tools

# Update plugin
/plugin marketplace update keyvalue-tools
/plugin update xclaude-dev-tools@keyvalue-tools

# Uninstall
/plugin uninstall xclaude-dev-tools@keyvalue-tools
/plugin marketplace remove keyvalue-tools

# Validate marketplace
claude plugin validate .
```
