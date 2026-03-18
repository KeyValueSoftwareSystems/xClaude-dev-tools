Review the current git diff (both staged and unstaged) critically.

## Step 0: Read project conventions (REQUIRED)

Check if `CLAUDE.md` exists in the project root.

**If CLAUDE.md does NOT exist**, stop immediately and output:
> **⚠ CLAUDE.md not found.** This plugin requires a `CLAUDE.md` file in your project root that defines your project's conventions (tech stack, architecture, naming rules, etc.).
>
> Run **`/setup`** to auto-generate one from your codebase, then re-run this command.

Do NOT proceed without CLAUDE.md.

**If CLAUDE.md exists**, read it and extract:
- Tech stack, frameworks, and language
- Architecture patterns and module boundaries
- Naming conventions and coding standards
- Testing requirements and conventions
- Any project-specific rules or anti-patterns to watch for

Apply these conventions throughout the review alongside the universal checks below.

## 1. Correctness
- Logic errors, off-by-one, null/nil/undefined handling, race conditions
- Incorrect use of framework APIs or language idioms
- Resource leaks (unclosed connections, file handles, streams)
- Concurrency issues (shared mutable state, missing synchronization)

## 2. Security
- Hardcoded secrets, credentials, API keys, or PII
- Injection vulnerabilities (SQL, XSS, command injection, template injection)
- Missing input validation on API endpoints or user-facing inputs
- Unprotected endpoints or missing authorization checks
- Sensitive data in logs or error responses
- Insecure deserialization or unsafe type casting

## 3. Architecture & Boundaries
- Violations of the project's module/layer boundaries (as defined in CLAUDE.md)
- Importing implementation details instead of interfaces across boundaries
- Circular dependencies between modules
- Side effects in unexpected places (e.g., network calls inside transactions)

## 4. Testing Gaps
- New production code without corresponding tests
- Tests with weak assertions (only checking for no errors, only null checks)
- Missing error/boundary case tests
- Test naming not following project conventions

## 5. Logging & Observability
- Missing logging on error paths (catch blocks without log statements)
- Sensitive data being logged (passwords, tokens, PII)
- Incorrect log levels
- Missing correlation/trace context for async flows

## 6. Naming & Clarity
- Violations of project naming conventions (as defined in CLAUDE.md)
- Variables/methods that don't clearly communicate intent
- Misleading names that suggest different behavior than implemented

## 7. Unnecessary Changes
- Formatting-only diffs mixed with functional changes
- Unrelated refactors bundled in
- Debug code left in (print statements, commented-out code, TODO/FIXME without tickets)

---

For each issue found, output:

```
[SEVERITY] file:line — description
  Fix: suggested fix
```

Severity levels:
- **CRITICAL** — Must fix before merge (security, correctness, data loss risk)
- **WARNING** — Should fix (testing gaps, architecture violations, missing docs)
- **SUGGESTION** — Nice to have (naming, clarity, style)

Run: `git diff` and `git diff --cached` to review all changes.
