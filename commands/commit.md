---
description: Check changes, create commit, and push to your feature branch
argument-hint: Optional commit message override (leave empty for AI-generated message)
---

You are a git workflow assistant. Your task is to review changes, create per-project commits for this monorepo, and push to feature branches for separate PRs.

## Step 0: Get User's Branch Prefix

Use **AskUserQuestion** to ask the user for their branch name prefix:

```
What is your branch name prefix? (e.g., your name or username)
This will be used for branch names like: <prefix>/admin-api-integration
```

Options to present:
- Common team member names as options if known from recent git history
- Allow custom input

Alternatively, check recent branches to suggest the user's typical prefix:
```bash
git branch -r | grep -oE '[^/]+/[^/]+' | cut -d'/' -f2 | sort | uniq -c | sort -rn | head -5
```

Store the result as `$USER_PREFIX` for use throughout this workflow (e.g., `lukas`, `dongsub`, `john`).

## Step 1: Check for Submodule Changes (IMPORTANT)

Check if `.claude` is a submodule with uncommitted changes:

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

## Step 2: Detect Projects with Changes

This is a monorepo with multiple projects. Detect which projects have changes:

```bash
git status --porcelain
```

Group the changes by project folder. Known project folders are:
- `backend` - NestJS backend API
- `frontend` - Legacy frontend (deprecated)
- `frontend-admin-dashboard` - Admin dashboard React app
- `frontend-coach-dashboard` - Coach dashboard React app
- `.claude-project` - Project documentation and plans

For each project with changes, count:
- Modified files (M)
- New/untracked files (??)
- Deleted files (D)

## Step 3: Present Workflow Selection

Use **AskUserQuestion** to let the user choose the commit/PR strategy:

Show the user a summary like:
```
Projects with changes detected:
1. frontend-admin-dashboard (15 files: 12 modified, 3 new)
2. frontend-coach-dashboard (17 files: 1 modified, 16 new)
3. .claude-project (2 files)
```

Options to present:
1. **"Separate branches & PRs"** (Recommended) - Create feature branch per project (e.g., `$USER_PREFIX/admin-api-integration`), each gets its own PR
2. **"Single branch, separate commits"** - All commits go to `$USER_PREFIX` branch, one PR with organized commits
3. **"Single branch, single commit"** - Original behavior, one commit for everything

If there's only ONE project with changes, skip to single project workflow.

## Step 4: Branch Strategy Per Project

### For "Separate branches & PRs" workflow:

For EACH project with changes:

1. **Stash other changes temporarily** (to keep working directory clean):
   ```bash
   git stash push -m "temp-stash-for-commit" -- <other-project-folders>
   ```

2. **Create/checkout feature branch from dev:**
   ```bash
   git checkout dev
   git pull origin dev
   git checkout -b $USER_PREFIX/<project-short-name>-<feature-description>
   ```

   Branch naming examples (where $USER_PREFIX is the user's name/prefix):
   - `$USER_PREFIX/admin-api-integration`
   - `$USER_PREFIX/coach-e2e-tests`
   - `$USER_PREFIX/backend-auth-fix`
   - `$USER_PREFIX/docs-update`

3. **Stage and commit project files** (see Step 5)

4. **Push branch and create PR** (see Step 6)

5. **Return to original branch and restore stash:**
   ```bash
   git checkout <original-branch>
   git stash pop
   ```

6. **Repeat for next project**

### For "Single branch" workflows:

Create or checkout the user's branch and proceed to Step 5:
```bash
git checkout -b $USER_PREFIX || git checkout $USER_PREFIX
```

## Step 5: Stage and Commit Per Project

For EACH project being committed:

### Stage only that project's files:
```bash
git add "<project-folder>/"
```

For root-level files like `.claude-project`:
```bash
git add ".claude-project/"
```

### Do NOT stage:
- `.env` files or files containing secrets
- `credentials.json` or similar sensitive files
- `node_modules/`, `dist/`, build artifacts
- `playwright-report/`, `test-results/` (test output)

### Create a project-scoped commit:

Generate a commit message with project scope:

```
<type>(<scope>): <concise description>

<optional body explaining the "why">

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

**Scope abbreviations:**
- `admin` for frontend-admin-dashboard
- `coach` for frontend-coach-dashboard
- `backend` for backend
- `docs` for .claude-project or documentation

**Type prefixes:**
- `feat:` - New feature
- `fix:` - Bug fix
- `refactor:` - Code refactoring
- `docs:` - Documentation
- `test:` - Tests
- `chore:` - Maintenance tasks
- `style:` - Formatting, no code change

**Examples:**
- `feat(admin): Add user management API integration`
- `feat(coach): Add Playwright E2E tests for dashboard pages`
- `fix(backend): Resolve authentication token expiry issue`
- `docs: Update API integration plan`

If $ARGUMENTS is provided by the user, use it as the commit message for the FIRST project only.

Use a HEREDOC for the commit message:
```bash
git commit -m "$(cat <<'EOF'
<message here>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## Step 6: Push and Create PRs

### For "Separate branches & PRs" workflow:

For EACH feature branch:

1. **Push the branch:**
   ```bash
   git push -u origin $USER_PREFIX/<branch-name>
   ```

2. **Create PR using gh CLI:**
   ```bash
   gh pr create --base dev --head $USER_PREFIX/<branch-name> --title "<PR title>" --body "$(cat <<'EOF'
   ## Summary
   <1-3 bullet points describing the changes>

   ## Project
   <project-name> (e.g., frontend-admin-dashboard)

   ## Changes
   - <list key changes>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

3. **Save the PR URL** to report at the end.

### For "Single branch" workflows:

1. **Push all commits to user's branch:**
   ```bash
   git push -u origin $USER_PREFIX
   ```

2. **Check if PR already exists:**
   ```bash
   gh pr view $USER_PREFIX --json number 2>/dev/null || echo "NO_PR"
   ```

3. **Create PR if none exists:**
   ```bash
   gh pr create --base dev --head $USER_PREFIX --title "<PR title summarizing all commits>" --body "$(cat <<'EOF'
   ## Summary
   <1-3 bullet points describing all changes>

   ## Projects Modified
   <list of projects with changes>

   ## Commits
   <list of commit summaries>

   🤖 Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

4. **If PR already exists, show the existing PR link:**
   ```bash
   gh pr view $USER_PREFIX --web
   ```

## Step 7: Report Results

### For "Separate branches & PRs":

```
✓ Created 2 feature branches with PRs:

1. $USER_PREFIX/admin-api-integration
   - Commit: abc1234 - feat(admin): Add user management API integration
   - PR: https://github.com/org/repo/pull/42
   - 15 files in frontend-admin-dashboard/

2. $USER_PREFIX/coach-e2e-tests
   - Commit: def5678 - feat(coach): Add Playwright E2E tests
   - PR: https://github.com/org/repo/pull/43
   - 12 files in frontend-coach-dashboard/

Current branch: <original-branch>
```

### For "Single branch" workflows:

```
✓ Created 2 commits on $USER_PREFIX branch:

1. abc1234 - feat(admin): Add user management API integration
   - 15 files in frontend-admin-dashboard/

2. def5678 - feat(coach): Add Playwright E2E tests
   - 12 files in frontend-coach-dashboard/

✓ Pushed to origin/$USER_PREFIX
✓ PR: https://github.com/org/repo/pull/44 (created/updated)
```

## Error Handling

- If user doesn't provide a prefix: Use a default or ask again
- If there are no changes: Inform user and stop
- If only one project has changes: Skip selection, use single branch workflow
- If commit fails (pre-commit hooks): Fix issues and retry with a NEW commit
- If push fails: Check branch status and inform user
- If PR creation fails: Show the push URL and suggest manual PR creation
- If user cancels selection: Stop gracefully
- If stash fails: Warn user and suggest manual intervention

## Important Notes

- **User-provided prefix** - Branch names use the prefix the user provides (their name, username, etc.)
- **Separate PRs are independent** - They can be reviewed and merged separately
- **Branch naming** - Always prefix with `$USER_PREFIX/` for feature branches
- **After merging PRs** - Delete the feature branches to keep repo clean
- **.claude-project changes** - Can be included with any project or committed separately
