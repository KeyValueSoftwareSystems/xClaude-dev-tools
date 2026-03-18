Generate a well-structured commit for the current staged and unstaged changes.

## Step 0: Read project conventions (REQUIRED)

Check if `CLAUDE.md` exists in the project root.

**If CLAUDE.md does NOT exist**, stop immediately and output:
> **⚠ CLAUDE.md not found.** This plugin requires a `CLAUDE.md` file in your project root that defines your project's conventions (tech stack, naming rules, commit format, etc.).
>
> Run **`/setup`** to auto-generate one from your codebase, then re-run this command.

Do NOT proceed without CLAUDE.md.

**If CLAUDE.md exists**, read it and extract:
- Commit message format and conventions (e.g., conventional commits, prefixes, scopes)
- Any project-specific commit rules or guidelines

Use these conventions throughout.

## Step 1: Analyze changes

Run these in parallel:
- `git status` to see all modified/untracked files
- `git diff` and `git diff --cached` to see actual changes
- `git log --oneline -10` to match the repo's commit message style

## Step 2: Let user select files to commit

- List all modified, staged, and untracked files
- Flag any files that look like secrets (.env, credentials, tokens, API keys) with a warning
- Use the **AskUserQuestion tool** with `multiSelect: true` to let the user pick which files to include
  - Show each file as an option with its status (modified/new/deleted)
  - Pre-recommend files that look related to each other
- Stage ONLY the selected files by name (NOT `git add -A` or `git add .`)
- If no files are selected, stop

## Step 3: Draft commit message

Analyze the staged changes and draft a commit message following the project's conventions (from CLAUDE.md or git log style). If no convention is established, use:
- Format: `<type>(<scope>): <description>`
- Types: feat, fix, refactor, test, chore, docs, perf, ci
- Scope: the module or area changed (infer from directory structure and file paths)
- Description: concise, imperative mood, focuses on WHY not WHAT
- Add body (separated by blank line) only if the change needs explanation

## Step 4: Confirm with user

Output the summary as plain text, then use **AskUserQuestion tool** in the same response to confirm.

```
**Files to commit:**
- path/to/file1 (modified)
- path/to/file2 (new)

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
