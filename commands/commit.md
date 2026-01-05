---
description: Check changes, create commit, push to lukas, and create PR to dev
argument-hint: Optional commit message override (leave empty for AI-generated message)
---

You are a git workflow assistant. Your task is to review changes, create a commit, push to the `lukas` branch, and create a PR to `dev`.

## Step 1: Review Changes

Run these commands in parallel to understand the current state:

1. `git status` - See all staged/unstaged/untracked files
2. `git diff` - See unstaged changes
3. `git diff --cached` - See staged changes
4. `git log --oneline -5` - See recent commit message style

## Step 2: Analyze and Confirm

Review the changes and identify:
- What files were modified/added/deleted
- The nature of the changes (feature, fix, refactor, docs, etc.)
- Any files that should NOT be committed (secrets, local configs, etc.)

If there are no changes to commit, inform the user and stop.

## Step 3: Stage Changes

Stage all relevant files:
- Use `git add .` for all changes, or
- Selectively add files if some should be excluded

Do NOT stage:
- `.env` files or files containing secrets
- `credentials.json` or similar sensitive files
- Files the user specifically asked to exclude

## Step 4: Create Commit

Generate a commit message following this format:

```
<type>: <concise description>

<optional body explaining the "why">

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

Type prefixes:
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code refactoring
- `docs:` - Documentation
- `test:` - Tests
- `chore:` - Maintenance tasks
- `style:` - Formatting, no code change

If $ARGUMENTS is provided, use it as the commit message instead of generating one.

Use a HEREDOC for the commit message:
```bash
git commit -m "$(cat <<'EOF'
<message here>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## Step 5: Push to Remote

Push the commit to the lukas branch:
```bash
git push origin lukas
```

If the push fails due to upstream issues, try:
```bash
git push -u origin lukas
```

## Step 6: Create Pull Request

Create a PR from `lukas` to `dev`:

```bash
gh pr create --base dev --head lukas --title "<PR title>" --body "$(cat <<'EOF'
## Summary
<bullet points summarizing changes>

## Test plan
- [ ] <testing checklist items>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

The PR title should match or be similar to the commit message.

## Step 7: Report Results

After completion, report:
- The commit SHA and message
- The PR URL
- Any warnings or issues encountered

## Error Handling

- If there are no changes: Inform user and stop
- If commit fails (pre-commit hooks): Fix issues and retry with a NEW commit
- If push fails: Check branch status and inform user
- If PR already exists: Provide the existing PR URL instead
