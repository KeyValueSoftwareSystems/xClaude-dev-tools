Generate a well-structured commit for the current staged and unstaged changes.

## Step 1: Analyze changes

Run these in parallel:
- `git status` to see all modified/untracked files
- `git diff` and `git diff --cached` to see actual changes
- `git log --oneline -10` to match the repo's commit message style

## Step 2: Let user select files to commit

- List all modified, staged, and untracked files
- Flag any files that look like secrets (.env, credentials, tokens) with a warning
- Use the **AskUserQuestion tool** with `multiSelect: true` to let the user pick which files to include
  - Show each file as an option with its status (modified/new/deleted)
  - Pre-recommend files that look related to each other
- Stage ONLY the selected files by name (NOT `git add -A` or `git add .`)
- If no files are selected, stop

## Step 3: Draft commit message

Analyze the staged changes and draft a commit message:
- Use conventional commit format: `<type>(<scope>): <description>`
- Types: feat, fix, refactor, test, chore, docs, perf, ci
- Scope: the module or area changed (e.g., common, customer, customerapi, build)
- Description: concise, imperative mood, focuses on WHY not WHAT
- Add body (separated by blank line) only if the change needs explanation

Examples:
```
feat(customer): add step-up authentication filter

Adds @StepUpAuth annotation and filter that validates elevated JWT
claims before granting access to sensitive endpoints.
```
```
fix(common): prevent race condition in operation ID consumption
```
```
test(common): add unit tests for AwsSqsService message attributes
```

## Step 4: Confirm with user

Output the summary as plain text, then use **AskUserQuestion tool** in the same response to confirm.

```
**Files to commit:**
- path/to/file1.java (modified)
- path/to/file2.java (new)

**Proposed commit message:**

type(scope): description

Optional body explaining the change.
```

Then immediately call **AskUserQuestion tool** with a simple approve/edit question.
Do NOT put the commit message inside the AskUserQuestion — it must be visible as regular text output above the question.

## Step 5: Commit

Create the commit using a HEREDOC:
```bash
git commit -m "$(cat <<'EOF'
<commit message here>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

## Step 6: Offer to push

After a successful commit, use **AskUserQuestion tool** to ask if the user wants to push to the remote branch.

Options:
- **Yes, push** — push to the current branch's upstream. If no upstream is set, use `git push -u origin <current-branch-name>`.
- **No, done** — stop here.

If the user chooses to push, run the appropriate `git push` command and confirm the result.

## Guardrails
- NEVER use `git add -A` or `git add .`
- NEVER commit .env, credentials, or secret files — warn the user if they're in the diff
- NEVER amend a previous commit unless the user explicitly asks
- ALWAYS confirm with the user before running `git commit`
- If there are no changes to commit, say so and stop
