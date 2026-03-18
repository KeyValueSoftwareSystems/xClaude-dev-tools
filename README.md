# xClaude Dev Tools

A Claude Code plugin with slash commands for development workflows. Works with **any tech stack** — the plugin reads your project's `CLAUDE.md` to understand conventions and adapts accordingly.

## Getting Started

### 1. Install the plugin

**Test locally:**
```bash
claude --plugin-dir /path/to/xclaude-dev-tools
```

**Install from marketplace:**
```
/plugin install xclaude-dev-tools
```

### 2. Run `/setup` (required, one-time)

Every command in this plugin requires a `CLAUDE.md` file in your project root. This file tells the plugin about your project's tech stack, conventions, and rules.

```
/xclaude-dev-tools:setup
```

This analyzes your codebase and generates a `CLAUDE.md` with:
- Tech stack and framework details
- Architecture and module boundaries
- Naming conventions and coding standards
- Testing framework, naming, and assertion style
- Database migration tool and conventions
- Commit message format

Review the generated file, tweak as needed, and you're ready to go.

> **Why?** A generic plugin can't know your project uses Flyway vs Alembic, JUnit vs Jest, or that your services must end with `Service`. `CLAUDE.md` bridges that gap — it's the single source of truth for your project's conventions, and every command reads it before doing anything.

## Commands

| Command | Description |
|---------|-------------|
| `/setup` | **Run first.** Analyzes codebase and generates `CLAUDE.md` |
| `/generate-commit` | Interactively stage files and create well-structured commits |
| `/review` | Review current git diff for correctness, security, architecture issues |
| `/pr-review` | Review a GitHub PR using the same review criteria |
| `/pull-request` | Create or update a GitHub PR with a structured description |
| `/sync-pr` | Lightweight PR description refresh after new commits |
| `/migration-review` | Review database migrations for safety and best practices |
| `/test-for` | Generate unit tests for a file/class following project conventions |

## Usage

Once installed, commands are available as:

```
/xclaude-dev-tools:setup
/xclaude-dev-tools:generate-commit
/xclaude-dev-tools:review
/xclaude-dev-tools:pr-review
/xclaude-dev-tools:pull-request
/xclaude-dev-tools:sync-pr
/xclaude-dev-tools:migration-review
/xclaude-dev-tools:test-for <ClassName>
```

If you run any command without a `CLAUDE.md`, it will stop and prompt you to run `/setup` first.

## How `CLAUDE.md` Works

The `CLAUDE.md` file lives in your project root and looks like this (example for a Spring Boot project):

```markdown
# Project: Customer API

A Spring Boot 3.2 REST API for customer management.

## Tech Stack
- Java 17+ / Spring Boot 3.2.5
- Spring Data JPA + H2 (dev) / PostgreSQL (prod)
- Maven

## Architecture
- Module structure: com.example.demo.<domain>
- Layers: Controller → Service → Repository

## Conventions
- @Service classes must end with Service
- Use constructor injection (no @Autowired)
- Commit format: <type>(<scope>): <description>

## Testing
- JUnit 5 + Mockito
- Test naming: methodName_condition_expectedResult

## Database Migrations
- Flyway SQL in src/main/resources/migrations/
- Naming: V<number>__<description>.sql
```

Every command reads this file and adapts its behavior. A React project would have different conventions, and the same commands would work accordingly.

## Development

```bash
# Load plugin during development
claude --plugin-dir ./xclaude-dev-tools

# Reload after making changes (inside Claude Code)
/reload-plugins

# Debug plugin loading
claude --debug --plugin-dir ./xclaude-dev-tools
```
