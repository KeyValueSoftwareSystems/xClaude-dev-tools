Review the current git diff (both staged and unstaged) critically across these dimensions:

## 1. Correctness
- Logic errors, off-by-one, null handling, race conditions
- Incorrect use of Spring annotations or framework APIs
- Transaction boundary issues (network calls inside @Transactional)

## 2. Security
- Hardcoded secrets, credentials, or PII
- SQL injection, XSS, or other injection vulnerabilities
- Missing input validation on API endpoints (@Valid, Bean Validation)
- Unprotected endpoints (missing security configuration)
- Sensitive data in logs or error responses

## 3. API Documentation & Standards
- Missing or incomplete API documentation annotations on REST endpoints
- Missing request validation annotations (@NotNull, @Size, etc.)
- Inconsistent error response format

## 4. Architecture & Module Boundaries
- Undeclared cross-module dependencies
- Importing implementation classes instead of interfaces from other modules
- Concrete classes in named interface packages (should be interfaces only)
- Domain modules depending on other domain modules
- Direct service calls between modules instead of events
- Network calls inside @Transactional methods (must use outbox pattern)

## 5. Testing Gaps
- New production code without corresponding tests
- Tests with weak assertions (assertDoesNotThrow, assertNotNull only)
- Missing error/boundary case tests
- Test naming not following `methodName_condition_expectedResult` convention

## 6. Logging
- Missing logging on error paths (catch blocks without log statements)
- Sensitive data logged (passwords, tokens, PII, full account numbers)
- Incorrect log levels (using INFO for debug-level detail, or DEBUG for errors)
- Missing correlation ID / trace context in log statements for async flows
- Using System.out.println instead of SLF4J logger (@Slf4j)

## 7. Naming & Clarity
- @Service classes not ending with "Service"
- @Repository classes not ending with "Repository"
- @Controller classes not ending with "Controller"
- Variables/methods that don't clearly communicate intent

## 8. Unnecessary Changes
- Formatting-only diffs mixed with functional changes
- Unrelated refactors bundled in
- Debug code left in (System.out.println, commented-out code)

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
