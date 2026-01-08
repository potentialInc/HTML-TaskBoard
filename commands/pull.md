---
description: Pull latest changes from remote for parent repo and all submodules
argument-hint: Optional remote name (default: origin)
---

You are a git workflow assistant. Your task is to pull the latest changes from remote for both the parent repository and all git submodules.

## Step 1: Check Git Status

First, check the current state of the repository:

```bash
git status
```

Review the output:
- If there are uncommitted changes (staged or unstaged), warn the user that pulling may cause conflicts
- If the working directory is clean, proceed to the next step

## Step 2: Pull Parent Repository

Pull the latest changes from remote:

```bash
git pull origin
```

If $ARGUMENTS is provided (e.g., a different remote name), use it:
```bash
git pull $ARGUMENTS
```

Handle potential issues:
- If there are merge conflicts, inform the user and stop
- If the pull was successful, report what was updated

## Step 3: Initialize and Update Submodules

First, ensure all submodules are initialized:

```bash
git submodule init
```

Then update all submodules recursively:

```bash
git submodule update --init --recursive
```

## Step 4: Pull Latest in Each Submodule

Pull the latest changes from the `main` branch in each submodule:

```bash
git submodule foreach 'git checkout main && git pull origin main'
```

Note: This will:
- Checkout the main branch in each submodule
- Pull the latest changes from origin/main

## Step 5: Report Results

After completion, report:
- Whether the parent repo was updated (and what changed)
- Whether submodules were updated
- Any warnings or issues encountered

## Error Handling

- If working directory is dirty: Warn user but continue (let git handle conflicts)
- If pull fails due to conflicts: Stop and inform user how to resolve
- If submodule update fails: Report which submodule failed and why
- If no submodules exist: Skip submodule steps silently
