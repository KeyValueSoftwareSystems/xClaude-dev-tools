Review a GitHub Pull Request using the same review criteria defined in the `/review` command, but applied to a PR diff instead of the local git diff.

**Input**: `$ARGUMENTS`

## Step 0: Read project conventions (REQUIRED)

Check if `CLAUDE.md` exists in the project root.

**If CLAUDE.md does NOT exist**, stop immediately and output:
> **⚠ CLAUDE.md not found.** This plugin requires a `CLAUDE.md` file in your project root that defines your project's conventions (tech stack, architecture, naming rules, etc.).
>
> Run **`/setup`** to auto-generate one from your codebase, then re-run this command.

Do NOT proceed without CLAUDE.md.

**If CLAUDE.md exists**, read it and extract the project's tech stack, conventions, and standards. Apply these during the review.

## Step 1: Resolve the PR

If `$ARGUMENTS` is empty or blank:
- Run `gh pr list --state open --limit 20` to list open PRs
- Use the **AskUserQuestion tool** to ask the user which PR to review, showing the list

If `$ARGUMENTS` is a number, use it as the PR number directly.

## Step 2: Fetch PR metadata and diff

```bash
gh pr view <PR_NUMBER> --json title,body,baseRefName,headRefName,author,files,additions,deletions
gh pr diff <PR_NUMBER>
```

Announce: **Reviewing PR #<number>: <title>** by <author> (`<head>` → `<base>`, +<additions> -<deletions>)

## Step 3: Review the diff

Apply all review dimensions from the `/review` command to the PR diff:
1. Correctness
2. Security
3. Architecture & Boundaries (per CLAUDE.md conventions)
4. Testing Gaps
5. Logging & Observability
6. Naming & Clarity (per CLAUDE.md conventions)
7. Unnecessary Changes

For ambiguous diffs (new files, signature changes), read the full file for context.

## Step 4: Output the review

Use the same output format (`[SEVERITY] file:line — description`), then end with:

```
## Summary

**PR:** #<number> — <title>
**Verdict:** APPROVE / REQUEST CHANGES / COMMENT
**Issues:** X critical, Y warnings, Z suggestions

<1-2 sentence overall assessment>
```

**Guardrails**:
- Do NOT approve PRs with CRITICAL issues
- If the PR has no issues, say so clearly — don't invent problems
- Focus on substantive issues, not style nitpicks (unless they violate project conventions from CLAUDE.md)
- For large PRs (>500 lines changed), group findings by file
