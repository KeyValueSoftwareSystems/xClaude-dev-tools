Manage GitHub Pull Requests for the current branch — **create** a new PR or **update** an existing one.

Before doing anything, announce clearly to the user:

> **`/pull-request` — This command can create a new PR or update an existing PR's description for the current branch.**

**Input**: `$ARGUMENTS` (optional: base branch)

## Step 1: Resolve base branch

If `$ARGUMENTS` is provided, use it as the base branch directly.

If `$ARGUMENTS` is empty or blank:
- Run `git branch -r` to list remote branches
- Use the **AskUserQuestion tool** to ask the user which branch to raise the PR against, showing common options (e.g., develop, main, and any other relevant branches)

## Step 2: Check for existing PR

First, check if the PR is closed or merged:
- Run `gh pr list --head <current-branch> --state closed --json number,title,url,state,mergedAt` to check for closed/merged PRs
- If a **merged** PR exists, inform the user: "PR #<number> has already been merged." and stop
- If a **closed** (not merged) PR exists, inform the user: "PR #<number> was closed." and stop

Then check for an open PR:
- Run `gh pr list --head <current-branch> --state open --json number,title,url,body` to check if an open PR exists

**If an open PR exists**, switch to **update mode**:
- Announce: "Found existing PR #<number>: <title> — <url>"
- Gather context (Step 3) using the PR's base branch
- Compare the existing PR body against the current diff and commits
- Draft an updated title and body that reflects ALL commits (including new ones)
- Show the user a summary of what changed since the PR was created (new commits)
- Display the updated title and body as plain text, then use **AskUserQuestion tool** to ask:
  - **Update PR** — apply the new title and body via `gh pr edit <number> --title "..." --body "..."`
  - **Keep as is** — do nothing
  - **Edit first** — let the user modify before applying
- If the user chooses to update, run `gh pr edit` and return the PR URL
- Stop here — do NOT create a duplicate PR

**If no PR exists (open or closed)**, continue to Step 3.

## Step 3: Gather context

Run these in parallel:
- `git branch --show-current` to get the current branch
- `git log <base>...HEAD --oneline` to see all commits on this branch
- `git diff <base>...HEAD --stat` to see files changed summary
- `git diff <base>...HEAD` to see the full diff

## Step 4: Verify readiness

Check for issues before creating the PR:
- If there are uncommitted changes, warn the user and suggest `/generate-commit` first
- If the branch has no commits ahead of base, stop — nothing to PR
- If the branch is not pushed to remote, push it with `git push -u origin <branch>`
- Confirm the push with the user using **AskUserQuestion tool** before pushing

## Step 5: Analyze changes and draft PR

Read all commits and the full diff to understand the change. Then draft:

**Title:** Short, imperative, under 70 chars. Prefix with type if the branch name has one (e.g., `feat:`, `fix:`).

**Body:** Use this template:

```
## Summary
<3-5 bullet points describing what changed and why>

## Change Type
- [ ] New feature
- [ ] Bug fix
- [ ] Refactoring
- [ ] Config/infra change
- [ ] Documentation

## Risk Assessment
- **Blast radius**: <which modules are affected, what could break>
- **Rollback plan**: <how to revert if something goes wrong>
- **Data migration**: Yes/No

## Testing Evidence
- **Automated tests**: <new/modified test classes>
- **Edge cases covered**: <what boundary conditions are tested>

## API Changes (if applicable)
- [ ] API documentation annotations added/updated
- [ ] Security configuration reviewed
- [ ] Request validation added
- [ ] Error responses follow consistent format

## Checklist
- [ ] No secrets, credentials, or PII in diff
- [ ] No TODO/FIXME without a linked ticket
- [ ] No unrelated formatting changes mixed in
- [ ] PR is under 500 lines (if over, explain why)
- [ ] Database migrations are backward-compatible
- [ ] No network calls inside @Transactional methods
- [ ] Module boundaries respected

## AI Disclosure
- **Agent used**: Claude Code
- **Human review level**: <Reviewed and tested / Reviewed only / Generated>
```

Fill in the template based on the actual diff. Check the boxes that apply. Leave unchecked boxes that don't apply or need attention.

## Step 6: Confirm with user

Show the user the draft title and body. Use **AskUserQuestion tool** to confirm or let them edit.

## Step 7: Create the PR

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

Return the PR URL when done.

## Guardrails
- NEVER create a PR without confirming with the user first
- NEVER push to main/develop directly — PRs only
- If no tests are found in the diff, flag it prominently in the Testing Evidence section
- If the diff is over 500 lines, note it in the checklist and suggest splitting
