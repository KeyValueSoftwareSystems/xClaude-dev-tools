Quickly update an existing PR's description to reflect new commits — lightweight, low-token alternative to `/pull-request`.

Before doing anything, announce clearly to the user:

> **`/sync-pr` — Refreshes your PR description with new commits. Use `/pull-request` to create a new PR.**

**Input**: `$ARGUMENTS` (optional: PR number)

## Step 1: Check for uncommitted changes

Run `git status` to check for uncommitted or unstaged changes.

- If there are uncommitted changes, inform the user: "You have uncommitted changes. Run `/generate-commit` first, then push before running `/sync-pr`." and stop.
- If the branch is ahead of the remote (unpushed commits), inform the user: "You have unpushed commits. Push first with `git push`, then run `/sync-pr`." and stop.

## Step 2: Find the open PR

If `$ARGUMENTS` is a number, use it as the PR number.

Otherwise, run `gh pr list --head <current-branch> --state open --json number,title,url,baseRefName,body` to find the open PR for the current branch.

- If no open PR exists, inform the user: "No open PR found for this branch. Use `/pull-request` to create one." and stop.
- If an open PR exists, announce: "Syncing PR #<number>: <title> — <url>"

## Step 3: Gather lightweight context

Run these in parallel:
- `git log <base>...HEAD --oneline` to see ALL commits on this branch
- `git diff <base>...HEAD --stat` to see files changed summary (NOT the full diff)

Do NOT read the full diff — this is intentional to keep token usage low.

## Step 4: Update the description

Read the existing PR body. Then:
- Update the **Summary** section to reflect ALL commits (add bullets for new work, reword existing ones if needed)
- Update the **Testing Evidence** section if new test files appear in the stat
- Update the **Checklist** — re-check line count from stat
- Leave all other sections unchanged unless a commit clearly affects them

Do NOT rewrite the entire body from scratch. Patch only the sections that changed.

## Step 5: Confirm with user

Show the updated sections as plain text. Highlight what changed with a brief note like:
```
**Changes to PR description:**
- Summary: added 2 new bullets for commits abc1234, def5678
- Checklist: updated line count
```

Then show the full updated body.

Then immediately use **AskUserQuestion tool** in the same response to confirm:
- **Update PR** — apply changes
- **Keep as is** — do nothing

## Step 6: Apply the update

Run `gh api repos/{owner}/{repo}/pulls/<number> -X PATCH -f body="..."` to update the PR body.

Return the PR URL when done.

## Guardrails
- NEVER read the full diff — use stat only to keep token cost low
- NEVER create a new PR — only update existing ones
- NEVER rewrite sections that haven't changed
- If no new commits are found since the PR was created, inform the user and stop
