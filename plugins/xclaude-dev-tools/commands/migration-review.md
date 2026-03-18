Review database migration files for correctness, safety, and best practices.

**Input**: `$ARGUMENTS`

## Step 1: Identify migrations to review

If `$ARGUMENTS` is empty or blank:
- Run `git diff --name-only` and `git diff --cached --name-only` to find changed/new `.sql` files under `src/main/resources/migrations/`
- If no migration files in the diff, search `src/main/resources/migrations/` for the latest migration files (highest version numbers) across all schemas and review those
- Use the **AskUserQuestion tool** to confirm which migration files to review

Otherwise, locate the migration file(s) matching `$ARGUMENTS` (version number, filename, or schema name).

## Step 2: Read context

- Read the target migration file(s)
- Read adjacent migrations in the same schema (previous 2-3 versions) to understand existing table structure
- Identify which schema this migration belongs to (based on directory: `migrations/<schema>/`)

## Step 3: Review across these dimensions

### 1. Naming & Versioning
- File follows Flyway convention: `V<number>__<description>.sql`
- Version number is sequential (no gaps, no conflicts with existing versions)
- Description is descriptive and uses underscores (e.g., `V3__add_customer_email_index.sql`)

### 2. Backward Compatibility
- **Column drops**: Will existing application code break before/during deployment?
- **Column renames**: Should be a multi-step migration (add new → copy data → drop old) across releases
- **NOT NULL additions**: Existing rows must have a DEFAULT or a data migration
- **Type changes**: Could cause data loss or truncation?
- **Table drops**: Is the table truly unused? Check for references in the codebase

### 3. Data Safety
- Uses `IF NOT EXISTS` / `IF EXISTS` guards where appropriate
- Large table modifications use batched operations (not single ALTER on millions of rows)
- Data migrations handle NULL values and edge cases
- No destructive operations without a clear rollback path
- DELETEs or UPDATEs have appropriate WHERE clauses (no accidental full-table mutations)

### 4. Index Review
- New foreign keys have corresponding indexes
- Indexes support the query patterns (check repository methods in the codebase if needed)
- No duplicate or redundant indexes (subset of existing index)
- Partial indexes use appropriate WHERE clauses
- Large table index creation uses `CONCURRENTLY` where possible

### 5. Constraints & Types
- Primary keys are defined
- Foreign key constraints reference correct tables/columns with appropriate ON DELETE behavior
- CHECK constraints for domain-specific validations (e.g., positive amounts, valid status values)
- Appropriate column types (e.g., `TIMESTAMP WITH TIME ZONE` not `TIMESTAMP`, `NUMERIC` for money not `FLOAT`)
- NOT NULL on columns that should never be null

### 6. Schema Isolation
- Migration operates within its declared schema (matches directory name)
- No cross-schema references that would violate module boundaries
- Schema matches the module architecture (common, customer, audit, etc.)

### 7. Performance Impact
- Will this migration lock tables for extended periods?
- Large data migrations should be done in batches
- Adding indexes on large tables should use `CREATE INDEX CONCURRENTLY`
- Consider impact on running application during zero-downtime deployments

### 8. PostgreSQL Best Practices
- Uses PostgreSQL-appropriate types (UUID, JSONB, TIMESTAMPTZ, etc.)
- Comments on tables and columns for documentation (`COMMENT ON`)
- Avoids antipatterns (EAV tables, polymorphic associations without good reason)

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
