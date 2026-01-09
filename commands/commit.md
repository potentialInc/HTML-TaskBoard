---
description: Create commits on feature branches and create PRs (NEVER push directly to main)
argument-hint: Optional commit message override (leave empty for AI-generated message)
---

You are a git workflow assistant. Your task is to review changes, create per-project commits for this monorepo, and create PRs for code review.

## CRITICAL RULES (NEVER VIOLATE)

1. **NEVER push directly to `main` or `origin/main`** - All changes MUST go through PRs
2. **ALWAYS create a PR** - The workflow is NOT complete until a PR URL is generated
3. **STOP if PR creation fails** - Do NOT continue, do NOT suggest manual alternatives
4. **Use nested branch names** - `$USER_PREFIX/<feature-name>` (e.g., `lukas/admin-feature`)
5. **Separate PR per project folder** - Each modified project folder gets its own PR

---

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
git branch -r | grep -oE 'origin/[^/]+/' | sed 's/origin\///' | sed 's/\///' | sort | uniq -c | sort -rn | head -5
```

Store the result as `$USER_PREFIX` for use throughout this workflow (e.g., `lukas`, `dongsub`, `john`).

---

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

---

## Step 2: Detect Projects with Changes

Run `git status --porcelain` and group changes by their root folder.

### Project Folder Detection Rules:

**Known project patterns** (each gets its own PR):
- `backend/` - Backend API (NestJS, Django, etc.)
- `frontend/` - Main frontend app
- `frontend-*/` - Any frontend variant (e.g., `frontend-dashboard/`, `frontend-admin-dashboard/`, `frontend-coach-dashboard/`)
- `mobile/` - Mobile app
- `.claude-project/` - Project documentation

**Dynamic detection**: Any root-level folder with changes that:
- Contains `package.json` (Node.js project)
- Contains `requirements.txt` or `pyproject.toml` (Python project)
- Contains `go.mod` (Go project)
- Contains `Cargo.toml` (Rust project)

**Exclusions** (never create separate PRs for):
- Hidden folders (`.git/`, `.claude/`, `.vscode/`, etc.)
- `node_modules/`, `dist/`, `build/` (build artifacts)
- Root-level config files (should be grouped with related project or `.claude-project`)

### Example Output:
```
Projects with changes detected:
1. frontend-admin-dashboard/ (8 files: 5 modified, 3 new)
2. frontend-coach-dashboard/ (12 files: 10 modified, 2 deleted)
3. frontend-mobile/ (3 files: 3 new)
4. backend/ (5 files: 4 modified, 1 new)
5. .claude-project/ (2 files)
```

For each project with changes, count:
- Modified files (M)
- New/untracked files (??)
- Deleted files (D)

---

## Step 3: Present Workflow Selection

Use **AskUserQuestion** to let the user choose the commit/PR strategy:

Show the user a summary of detected projects, then present options:

**Options to present:**
1. **"Separate PRs per project"** (Recommended) - One feature branch and PR per project folder
2. **"Combined PR"** - Single feature branch with all changes, one PR

⚠️ **Both options MUST end with PR creation. There is no "push only" option.**

If there's only ONE project with changes, skip selection and proceed directly.

---

## Step 4: Branch Strategy Per Project

### For "Separate PRs per project" workflow:

For EACH project with changes:

1. **Stash other changes temporarily** (to keep working directory clean):
   ```bash
   git stash push -m "temp-stash-for-commit" -- <other-project-folders>
   ```

2. **Create/checkout feature branch from main:**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b $USER_PREFIX/<project-short-name>-<feature-description>
   ```

   Branch naming examples:
   - `lukas/admin-api-integration`
   - `dongsub/coach-e2e-tests`
   - `lukas/backend-auth-fix`
   - `lukas/mobile-navigation`
   - `lukas/docs-update`

3. **Stage and commit project files** (see Step 5)

4. **Push branch and create PR** (see Step 6)

5. **Return to main and restore stash:**
   ```bash
   git checkout main
   git stash pop
   ```

6. **Repeat for next project**

### For "Combined PR" workflow:

1. **Create single feature branch from main:**
   ```bash
   git checkout main
   git pull origin main
   git checkout -b $USER_PREFIX/<combined-feature-description>
   ```

2. **Stage and commit all changes** (can be multiple commits per project)

3. **Push branch and create single PR** (see Step 6)

---

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
- `mobile` for mobile or frontend-mobile
- `backend` for backend
- `docs` for .claude-project or documentation
- `dashboard` for frontend-dashboard

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
- `feat(mobile): Add navigation drawer component`
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

---

## Step 6: Push and Create PR (MANDATORY)

### For EVERY branch (this step is REQUIRED, not optional):

1. **Push the branch:**
   ```bash
   git push -u origin $USER_PREFIX/<branch-name>
   ```

   **If push fails due to conflict with flat `$USER_PREFIX` branch:**
   ```bash
   git push origin --delete $USER_PREFIX  # Delete the conflicting flat branch
   git push -u origin $USER_PREFIX/<branch-name>  # Retry
   ```

2. **Create PR using gh CLI (REQUIRED):**
   ```bash
   gh pr create --base main --head $USER_PREFIX/<branch-name> --title "<PR title>" --body "$(cat <<'EOF'
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

3. **VERIFY PR was created (REQUIRED):**
   ```bash
   gh pr view $USER_PREFIX/<branch-name> --json url --jq '.url'
   ```

   **If this command fails or returns empty: STOP immediately.**
   - Do NOT continue to the next project
   - Do NOT report success
   - Report the error and ask user to check GitHub access

4. **Store the PR URL** for the final report.

---

## Step 7: Report Results

The workflow is ONLY successful if ALL of these are true:
- ✓ Commits created on feature branch(es)
- ✓ Branch(es) pushed to origin
- ✓ PR(s) created with valid URLs

### Success Report Format:

```
✓ Workflow Complete

PR(s) Created:

1. https://github.com/org/repo/pull/42
   - Branch: lukas/admin-api-integration
   - Commit: abc1234 - feat(admin): Add user management API integration
   - Files: 15 in frontend-admin-dashboard/

2. https://github.com/org/repo/pull/43
   - Branch: lukas/coach-e2e-tests
   - Commit: def5678 - feat(coach): Add Playwright E2E tests
   - Files: 12 in frontend-coach-dashboard/

3. https://github.com/org/repo/pull/44
   - Branch: lukas/mobile-navigation
   - Commit: ghi9012 - feat(mobile): Add navigation drawer
   - Files: 3 in mobile/

Current branch: main
```

### Failure Report Format:

```
✗ Workflow Failed

Error: PR creation failed for branch lukas/admin-api-integration
Command: gh pr create --base main ...
Error message: <error details>

Action Required: Check GitHub access with `gh auth status` and try again.
```

---

## Error Handling

### STOP conditions (halt workflow immediately):
- **PR creation fails** → STOP, show error, do NOT suggest manual PR creation
- **Push fails** → STOP, show error, check branch status
- **`gh` CLI not authenticated** → STOP, instruct user to run `gh auth login`
- **No changes detected** → STOP, inform user

### Retry conditions:
- **Commit fails (pre-commit hooks)** → Fix issues and retry with NEW commit
- **Branch already exists** → Ask user: delete and recreate, or use unique name
- **Push conflict with flat branch** → Delete flat branch and retry

### NEVER do these:
- ❌ Push directly to `main` or `origin/main`
- ❌ Push without creating a PR
- ❌ Report "success" if PR was not created
- ❌ Suggest "manual PR creation" as an alternative
- ❌ Continue to next project if current PR creation failed

---

## Important Notes

- **User-provided prefix** - Branch names use the prefix the user provides (their name, username, etc.)
- **Separate PRs are independent** - They can be reviewed and merged separately
- **Branch naming** - Always use `$USER_PREFIX/<feature-name>` format
- **After merging PRs** - Delete the feature branches to keep repo clean
- **.claude-project changes** - Can be included with any project PR or committed separately
- **PR is mandatory** - The workflow does not complete without a PR URL
