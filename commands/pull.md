---
description: Pull latest changes from remote for parent repo and all submodules
argument-hint: Optional branch name for parent repo (will prompt if not provided)
---

You are a git workflow assistant. Your task is to pull the latest changes from remote for both the parent repository and all git submodules.

## Submodule Branch Configuration

The following submodules have fixed branches:
- `.claude/base` → `main` branch
- `.claude` → `main` branch

The parent repository branch should be specified by the user.

## Step 1: Check Git Status

First, check the current state of the repository:

```bash
git status
```

Review the output:
- If there are uncommitted changes (staged or unstaged), warn the user that pulling may cause conflicts
- If the working directory is clean, proceed to the next step

## Step 2: Determine Parent Branch

If $ARGUMENTS is provided, use it as the branch name.

If $ARGUMENTS is NOT provided, ask the user which branch to pull for the parent repository using AskUserQuestion tool. Common options:
- `main` - Main branch
- `dev` - Development branch
- Current branch (show the current branch name)

## Step 3: Pull Parent Repository

Pull the latest changes from the determined branch:

```bash
git pull origin <branch>
```

Handle potential issues:
- If there are merge conflicts, inform the user and stop
- If the pull was successful, report what was updated

## Step 4: Initialize and Update Submodules

First, ensure all submodules are initialized:

```bash
git submodule init
```

Then update all submodules recursively:

```bash
git submodule update --init --recursive
```

## Step 5: Pull Latest in Each Submodule

Pull the latest changes from the configured branches in each submodule:

For `.claude/base`:
```bash
cd .claude/base && git checkout main && git pull origin main
```

For `.claude`:
```bash
cd .claude && git checkout main && git pull origin main
```

Note: Always use `main` branch for submodules regardless of the parent repo branch.

## Step 6: Report Results

After completion, report:
- Parent repo: which branch was pulled and what changed
- `.claude/base`: whether updated from main
- `.claude`: whether updated from main
- Any warnings or issues encountered

## Error Handling

- If working directory is dirty: Warn user but continue (let git handle conflicts)
- If pull fails due to conflicts: Stop and inform user how to resolve
- If submodule update fails: Report which submodule failed and why
- If no submodules exist: Skip submodule steps silently
