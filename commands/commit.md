---
description: Check changes, create commit, and push to lukas branch
argument-hint: Optional commit message override (leave empty for AI-generated message)
---

You are a git workflow assistant. Your task is to review changes, create a commit, and push to the `lukas` branch.

## Step 0: Check for Submodule Changes (IMPORTANT)

First, check if `.claude` is a submodule with uncommitted changes:

```bash
git status
```

If you see `.claude (modified content)` or `.claude (new commits)`, the `.claude` folder is a submodule with changes that need to be committed FIRST.

### Submodule Workflow:

1. **Check submodule status:**
   ```bash
   cd .claude && git status && git diff
   ```

2. **If there are changes in the submodule, commit them:**
   ```bash
   cd .claude && git add -A && git commit -m "$(cat <<'EOF'
   <type>: <description of submodule changes>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
   EOF
   )"
   ```

3. **Push the submodule:**
   ```bash
   cd .claude && git push
   ```

4. **Return to parent and continue** - The parent repo will now show `.claude (new commits)` which gets committed with the rest of the changes.

If `.claude` is NOT a submodule (no `.claude/.git` directory), skip this step.

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

## Step 6: Report Results

After completion, report:
- The commit SHA and message
- The remote branch URL
- Any warnings or issues encountered

## Error Handling

- If there are no changes: Inform user and stop
- If commit fails (pre-commit hooks): Fix issues and retry with a NEW commit
- If push fails: Check branch status and inform user
