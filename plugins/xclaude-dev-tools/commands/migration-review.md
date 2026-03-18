Review database migration files for correctness, safety, and best practices.

**Input**: `$ARGUMENTS`

## Step 0: Read project conventions (REQUIRED)

Check if `CLAUDE.md` exists in the project root.

**If CLAUDE.md does NOT exist**, stop immediately and output:
> **⚠ CLAUDE.md not found.** This plugin requires a `CLAUDE.md` file in your project root that defines your project's conventions (tech stack, migration tool, database type, etc.).
>
> Run **`/setup`** to auto-generate one from your codebase, then re-run this command.

Do NOT proceed without CLAUDE.md.

**If CLAUDE.md exists**, read it and extract:
- Migration tool used (Flyway, Alembic, Knex, Django migrations, Prisma, ActiveRecord, etc.)
- Migration file location and naming conventions
- Database type (PostgreSQL, MySQL, SQLite, etc.)
- Schema organization (single schema, multi-schema, per-module)
- Any project-specific migration rules

Apply these conventions throughout the review.

## Step 1: Identify migrations to review

If `$ARGUMENTS` is empty or blank:
- Run `git diff --name-only` and `git diff --cached --name-only` to find changed/new migration files
- If no migration files in the diff, search the project's migration directory for the latest migration files and review those
- Use the **AskUserQuestion tool** to confirm which migration files to review

Otherwise, locate the migration file(s) matching `$ARGUMENTS` (version number, filename, or path).

## Step 2: Read context

- Read the target migration file(s)
- Read adjacent migrations (previous 2-3 versions) to understand existing table structure
- Identify the migration framework from file naming patterns and project config

## Step 3: Review across these dimensions

### 1. Naming & Versioning
- File follows the project's migration naming convention
- Version/sequence number is correct (no gaps, no conflicts)
- Description is clear and descriptive

### 2. Backward Compatibility
- **Column drops**: Will existing application code break before/during deployment?
- **Column renames**: Should be a multi-step migration (add new → copy data → drop old) across releases
- **NOT NULL additions**: Existing rows must have a DEFAULT or a data migration
- **Type changes**: Could cause data loss or truncation?
- **Table drops**: Is the table truly unused? Check for references in the codebase

### 3. Data Safety
- Uses appropriate guards (`IF NOT EXISTS` / `IF EXISTS`) where supported
- Large table modifications use batched operations (not single ALTER on millions of rows)
- Data migrations handle NULL values and edge cases
- No destructive operations without a clear rollback path
- DELETEs or UPDATEs have appropriate WHERE clauses (no accidental full-table mutations)

### 4. Index Review
- New foreign keys have corresponding indexes
- Indexes support the query patterns in the codebase
- No duplicate or redundant indexes
- Large table index creation uses concurrent/online operations where supported

### 5. Constraints & Types
- Primary keys are defined
- Foreign key constraints reference correct tables/columns with appropriate cascade behavior
- CHECK constraints for domain-specific validations where appropriate
- Appropriate column types for the database engine
- NOT NULL on columns that should never be null

### 6. Schema Isolation
- Migration operates within its declared scope
- No cross-boundary references that would violate the project's module architecture (per CLAUDE.md)

### 7. Performance Impact
- Will this migration lock tables for extended periods?
- Large data migrations should be done in batches
- Consider impact on running application during zero-downtime deployments

---

## Output format

For each issue found:

```
[SEVERITY] file:line — description
  Context: why this matters
  Fix: suggested fix
```

Severity levels:
- **CRITICAL** — Must fix before merge (data loss risk, backward incompatibility, missing constraints)
- **WARNING** — Should fix (missing indexes, performance concerns, missing guards)
- **SUGGESTION** — Nice to have (comments, naming, style)

At the end, provide a summary:
```
Summary: X critical, Y warnings, Z suggestions
Migration safe to apply: YES / NO / YES WITH CAUTION
```

If no issues found, confirm the migration looks good with a brief rationale.
