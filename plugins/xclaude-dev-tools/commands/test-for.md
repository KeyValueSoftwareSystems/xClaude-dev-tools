Generate tests for a file/class/module following project conventions.

**Input**: `$ARGUMENTS`

## Step 0: Read project conventions (REQUIRED)

Check if `CLAUDE.md` exists in the project root.

**If CLAUDE.md does NOT exist**, stop immediately and output:
> **⚠ CLAUDE.md not found.** This plugin requires a `CLAUDE.md` file in your project root that defines your project's conventions (tech stack, testing framework, naming rules, etc.).
>
> Run **`/setup`** to auto-generate one from your codebase, then re-run this command.

Do NOT proceed without CLAUDE.md.

**If CLAUDE.md exists**, read it and extract:
- Testing framework and tooling (JUnit, pytest, Jest, Go testing, etc.)
- Test file location conventions (e.g., `src/test/`, `__tests__/`, `*_test.go`, `*.spec.ts`)
- Test naming conventions
- Mocking approach and preferred libraries
- Assertion style preferences
- Any project-specific testing rules

## Step 1: Resolve the target

If `$ARGUMENTS` is empty or blank:
- Run `git diff --name-only HEAD~1` to find recently changed production files
- Filter to source files only (exclude test files, config files, migrations)
- Use the **AskUserQuestion tool** to let the user pick which file to generate tests for

If `$ARGUMENTS` is provided, search the codebase for a matching file (by name, class name, or file path).

## Step 2: Read the target and its context

- Read the target file fully
- Read its dependencies/imports to understand what needs mocking
- Read any interfaces or base classes it implements/extends
- Read any existing test file for this target (to avoid duplicating tests)
- Look at existing tests in the project to match the style and patterns used

## Step 3: Generate tests

Follow the project's testing conventions discovered in Step 0 and Step 2. Apply these universal principles:

**Required coverage per public method/function:**
1. At least one test with specific value assertions for valid input (happy path)
2. At least one test for an error/boundary case

**Assertion quality — use specific assertions, NOT weak ones:**
```
// BAD — tells you nothing when it fails
assertNotNull(result)
expect(result).toBeDefined()

// GOOD — tells you exactly what went wrong
assertEqual(result.name, "expected_name")
expect(result.items).toHaveLength(3)
```

**For each public method/function, generate:**
- Happy path test with concrete input values and specific output assertions
- Error/boundary test (null input, empty collections, invalid args, exception cases)
- If the method has side effects (calls external services), verify the interactions

## Step 4: Write the test file

- If a test file already exists, add only the missing tests (do not overwrite existing ones)
- If no test file exists, create a new one at the correct path per project conventions
- Add all necessary imports
- Match the existing test style in the project

## Guardrails
- Do NOT generate tests for private/internal methods — test them through public interfaces
- Do NOT use weak assertions (assertNotNull, assertDoesNotThrow, toBeDefined) as the sole assertion
- Every test MUST have at least one meaningful assertion on a specific value
- If the target has no public methods/functions worth testing (e.g., pure config), say so and stop
- Follow the project's test runner and framework — do not introduce a different testing library
