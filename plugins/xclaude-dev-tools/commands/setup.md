Generate a `CLAUDE.md` file for this project by analyzing the codebase.

## Step 1: Check if CLAUDE.md already exists

If a `CLAUDE.md` already exists in the project root:
- Read it and display its contents
- Use **AskUserQuestion tool** to ask: "CLAUDE.md already exists. What would you like to do?"
  - **Regenerate** — start fresh (will overwrite)
  - **Review & update** — suggest improvements to the existing file
  - **Cancel** — stop
- If the user chooses "Review & update", analyze the codebase (Step 2) and suggest additions/corrections to the existing file, then apply edits after confirmation
- If the user chooses "Cancel", stop

## Step 2: Analyze the codebase

Run these in parallel to detect the project's tech stack and conventions:

**Package/build files:**
- Look for: `pom.xml`, `build.gradle`, `package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, `*.csproj`, `composer.json`
- Extract: language, framework, dependencies, build tool

**Project structure:**
- Run `find . -type f -name "*.java" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.go" -o -name "*.rs" -o -name "*.rb" -o -name "*.cs" | head -50` to identify the primary language
- Run `ls -d */` at root to understand top-level directory structure
- Look for `src/main/`, `src/`, `app/`, `lib/`, `cmd/`, `internal/` to detect architecture patterns

**Testing setup:**
- Look for test directories: `src/test/`, `__tests__/`, `tests/`, `test/`, `spec/`
- Read 2-3 existing test files to detect: framework, naming convention, assertion style, mocking approach

**Database/migrations:**
- Look for migration directories: `migrations/`, `db/migrate/`, `alembic/`, `prisma/migrations/`
- Detect migration tool from file naming patterns

**Existing conventions:**
- Read 3-5 source files to detect: naming patterns, import style, code organization
- Check for linter configs: `.eslintrc`, `pylint.ini`, `.golangci.yml`, `checkstyle.xml`
- Check for formatter configs: `.prettierrc`, `rustfmt.toml`, `.editorconfig`
- Read `git log --oneline -10` to detect commit message conventions

## Step 3: Draft CLAUDE.md

Based on the analysis, generate a `CLAUDE.md` following this structure:

```markdown
# Project: <project name>

<One-line description of what this project is>

## Tech Stack
- <Language version> / <Framework version>
- <Key libraries and their purpose>
- <Build tool>

## Architecture
- **Structure**: <how the code is organized — modules, packages, layers>
- <Key architectural patterns: MVC, hexagonal, microservices, monolith, etc.>
- <Important boundaries or rules about dependencies between modules>

## Conventions
- <Naming rules for classes, functions, files>
- <Import/dependency rules>
- <Framework-specific idioms the project follows>
- <Logging approach>
- Commit format: <detected format from git log>

## Testing
- <Test framework and runner>
- <Test file location and naming convention>
- <Mocking/stubbing approach>
- <Assertion style>
- <Minimum coverage expectations if any>

## Database Migrations (if applicable)
- <Migration tool>
- <File location and naming convention>
- <Any migration-specific rules>

## API (if applicable)
- <Base path or routing conventions>
- <Request/response patterns>
- <Validation approach>
```

Only include sections that are relevant. If the project has no database, omit that section. If it's not an API, omit that section.

## Step 4: Confirm with user

Display the drafted CLAUDE.md as plain text output.

Then use **AskUserQuestion tool** to ask:
- **Looks good, create it** — write the file
- **Edit first** — let the user suggest changes before writing
- **Cancel** — don't create

## Step 5: Write the file

Write `CLAUDE.md` to the project root.

Announce:
> **CLAUDE.md created.** All xclaude-dev-tools commands (`/review`, `/generate-commit`, `/test-for`, etc.) will now use these conventions automatically.

## Guardrails
- NEVER invent conventions that aren't evidenced in the codebase — only document what you can verify
- If you can't determine a convention with confidence, mark it with `<!-- TODO: verify -->` so the user knows to check
- Keep it concise — CLAUDE.md should be a quick reference, not exhaustive documentation
- Do NOT include information that changes frequently (current sprint, ticket numbers, deployment dates)
