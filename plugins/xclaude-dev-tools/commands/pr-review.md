Review a GitHub Pull Request using the same review criteria defined in `.claude/commands/review.md`, but applied to a PR diff instead of the local git diff.

**Input**: `$ARGUMENTS`

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

Read `.claude/commands/review.md` and apply all review dimensions listed there to the PR diff. For ambiguous diffs (new classes, signature changes), read the full file for context.

## Step 4: Output the review

Use the same output format from `review.md` (`[SEVERITY] file:line — description`), then end with:

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
