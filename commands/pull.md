---
description: Pull latest changes from remote for current branch and all submodules
argument-hint: Optional branch name override (default: current branch)
---

You are a git workflow assistant. Your task is to pull the latest changes from remote for the **current branch** in both the parent repository and all git submodules.

## Key Principle

**Pull to current branch** - Do NOT switch branches. Pull updates for whatever branch each repo is currently on.

## Submodule Architecture

This project uses nested submodules:

```
project/                    # Parent repo (current branch)
├── .claude/                # Submodule → project-claude (current branch)
│   ├── base/               # Submodule → claude-base (current branch)
│   ├── <tech-stack>/       # Submodules → claude-nestjs, claude-react, etc. (current branch)
│   └── commands -> base/commands
```

**Branch Policy:**
- **All repos:** Pull to current branch (do NOT switch branches)
- **Submodules:** Pull whatever branch they're currently on

**Important:** Update submodules in order from deepest to shallowest.

---

## Step 0: Check Current State

Before pulling, check the current state of all repos:

```bash
CURRENT_BRANCH=$(git branch --show-current)
echo "Parent repo branch: $CURRENT_BRANCH"
git status
```

If there are uncommitted changes (staged or unstaged), warn the user that pulling may cause conflicts.

---

## Step 1: Pull Parent Repository (Current Branch)

Pull the current branch without switching:

```bash
CURRENT_BRANCH=$(git branch --show-current)

if [ -z "$CURRENT_BRANCH" ]; then
  echo "⚠️ Parent repo is in detached HEAD state"
  # Cannot pull in detached HEAD - inform user
else
  echo "Pulling $CURRENT_BRANCH..."
  git pull origin "$CURRENT_BRANCH"
fi
```

If $ARGUMENTS is provided, use it as the branch name:
```bash
git checkout "$ARGUMENTS"
git pull origin "$ARGUMENTS"
```

Handle potential issues:
- If there are merge conflicts, inform the user and stop
- If the pull was successful, report what was updated

---

## Step 2: Initialize and Update Submodules

First, ensure all submodules are initialized:

```bash
git submodule init
git submodule update --init --recursive
```

This syncs submodules to the commits recorded in the parent repo.

---

## Step 3: Pull Nested Submodules (Deepest First)

**IMPORTANT:** Update submodules from deepest to shallowest to avoid conflicts.

### 3.1 Pull All Nested Submodules in .claude

```bash
cd .claude

# Find and pull all nested submodules
for submodule in */; do
  submodule_name="${submodule%/}"

  # Skip if not a git submodule
  if [ ! -e "$submodule_name/.git" ]; then
    continue
  fi

  # Skip symlinks (like commands)
  if [ -L "$submodule_name" ]; then
    continue
  fi

  echo "Pulling .claude/$submodule_name..."
  cd "$submodule_name"

  NESTED_BRANCH=$(git branch --show-current)
  if [ -z "$NESTED_BRANCH" ]; then
    echo "⚠️ .claude/$submodule_name is in detached HEAD, skipping pull"
  else
    echo "  Branch: $NESTED_BRANCH"
    git pull origin "$NESTED_BRANCH"
  fi

  cd ..
done

cd ..
```

### 3.2 Pull .claude Submodule

After nested submodules are pulled, pull `.claude` itself:

```bash
cd .claude

CLAUDE_BRANCH=$(git branch --show-current)
if [ -z "$CLAUDE_BRANCH" ]; then
  echo "⚠️ .claude is in detached HEAD, skipping pull"
else
  echo "Pulling .claude on branch: $CLAUDE_BRANCH"
  git pull origin "$CLAUDE_BRANCH"
fi

cd ..
```

**Note:** If `.claude` pull fails with "Entry would be overwritten by merge", it means a nested submodule has local changes. Resolve by:
1. Stashing or committing changes in the nested submodule
2. Or running `git submodule update --init --recursive --force` (loses local changes)

---

## Step 4: Check for Submodule Reference Updates

After pulling, check if the parent repo needs to record new submodule commits:

```bash
git status
```

If `.claude` shows as modified (new commits), inform the user:

```
Note: Submodules were updated. If you want to record these updates in the parent repo,
run `/commit` to create a commit with the updated submodule references.
```

---

## Step 5: Sync Commands from Submodules

After pulling submodule updates, verify commands are properly linked:

```bash
# Check if commands symlink exists and points to base/commands
if [ -L .claude/commands ]; then
  echo "Commands symlink OK: $(readlink .claude/commands)"
else
  echo "Warning: .claude/commands is not a symlink"
  # Recreate symlink if needed
  rm -rf .claude/commands
  ln -s base/commands .claude/commands
  echo "Recreated commands symlink"
fi
```

---

## Step 6: Report Results

After completion, report:

```
✓ Pull Complete

Parent repo:
  - Branch: <current-branch>
  - Status: <updated/already up to date>

Submodules updated:
  - .claude/base: <branch> - <status>
  - .claude/<other>: <branch> - <status>
  - .claude: <branch> - <status>

Commands: <symlink status>

Any issues: <warnings if any>
```

---

## Error Handling

| Issue | Resolution |
|-------|------------|
| Working directory is dirty | Warn user, continue (let git handle conflicts) |
| Pull fails due to conflicts | STOP, inform user how to resolve |
| Submodule in detached HEAD | Skip pull for that submodule, inform user |
| Submodule pull fails | Report which submodule failed and why |
| "Entry would be overwritten" | Nested submodule has uncommitted changes |
| No submodules exist | Skip submodule steps silently |
| Commands symlink broken | Recreate symlink to base/commands |

---

## Quick Reference

Pull order (deepest first):
1. `.claude/<all-nested-submodules>` → current branch
2. `.claude` → current branch
3. Parent repo → current branch (or user-specified)

---

## Alternative: Pull Specific Branch

To pull a specific branch instead of current:

```
/pull dev
```

This will:
1. Checkout `dev` in parent repo
2. Pull from `dev`
3. Update submodules (still use their current branches)
