# xClaude Dev Tools

A Claude Code plugin with slash commands for development workflows.

## Commands

| Command | Description |
|---------|-------------|
| `generate-commit` | Interactively stage files and create well-structured commits |
| `review` | Review current git diff for correctness, security, architecture issues |
| `pr-review` | Review a GitHub PR using the same review criteria |
| `pull-request` | Create or update a GitHub PR with a structured description |
| `sync-pr` | Lightweight PR description refresh after new commits |
| `migration-review` | Review database migrations for safety and best practices |
| `test-for` | Generate unit tests for a class following project conventions |

## Installation

### Test locally

```bash
claude --plugin-dir /path/to/xclaude-dev-tools
```

### Install from marketplace

```
/plugin install xclaude-dev-tools
```

## Usage

Once installed, commands are available as:

```
/xclaude-dev-tools:generate-commit
/xclaude-dev-tools:review
/xclaude-dev-tools:pr-review
/xclaude-dev-tools:pull-request
/xclaude-dev-tools:sync-pr
/xclaude-dev-tools:migration-review
/xclaude-dev-tools:test-for <ClassName>
```

## Development

```bash
# Load plugin during development
claude --plugin-dir ./xclaude-dev-tools

# Reload after making changes (inside Claude Code)
/reload-plugins

# Debug plugin loading
claude --debug --plugin-dir ./xclaude-dev-tools
```
