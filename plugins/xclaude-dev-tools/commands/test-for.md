Generate tests for a class following project conventions.

**Input**: `$ARGUMENTS`

## Step 1: Resolve the target class

If `$ARGUMENTS` is empty or blank:
- Run `git diff --name-only HEAD~1` to find recently changed production files
- Filter to `src/main/java/**/*.java` only
- Use the **AskUserQuestion tool** to let the user pick which class to generate tests for

If `$ARGUMENTS` is provided, search the codebase for a matching class (by class name or file path).

## Step 2: Read the class and its context

- Read the target class fully
- Read its constructor dependencies to understand what needs mocking
- Read any interfaces it implements
- Read any existing test file for this class (to avoid duplicating tests)
- Check the test naming convention from existing tests in the project

## Step 3: Generate tests

Follow these project conventions (from CLAUDE.md):

**Test class setup:**
- `@ExtendWith(MockitoExtension.class)`
- `@Mock` for all constructor dependencies
- `@InjectMocks` for the class under test
- Place in the matching test package under `src/test/java/`

**Test naming:** `methodName_condition_expectedResult`

**Required coverage per method:**
1. At least one test with specific value assertions for valid input
2. At least one test for an error/boundary case

**Assertion style — use specific assertions, NOT weak ones:**
```java
// BAD
assertDoesNotThrow(() -> service.process(input));
assertNotNull(result);

// GOOD
assertThat(result.total()).isEqualTo(BigDecimal.valueOf(30.0));
assertThat(result.lineItems()).hasSize(2);
assertEquals(MessageChannel.SQS, ex.getChannel());
```

**For each public method, generate:**
- Happy path test with concrete input values and specific output assertions
- Error/boundary test (null input, empty collections, exception cases)
- If the method has side effects (calls other services), verify interactions with `verify()`

## Step 4: Write the test file

- If a test file already exists, add only the missing tests (do not overwrite existing ones)
- If no test file exists, create a new one at the correct path
- Add all necessary imports

## Guardrails
- Do NOT generate tests for private methods — test them through public methods
- Do NOT use `assertDoesNotThrow` or `assertNotNull` as the sole assertion
- Do NOT create integration tests — keep them as unit tests with mocks
- Every test MUST have at least one meaningful assertion on a specific value
- If the class has no public methods worth testing (e.g., pure config), say so and stop
