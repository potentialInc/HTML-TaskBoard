---
description: Check submodule health and sync status
argument-hint: Optional --fix to auto-fix common issues
---

You are a git submodule health check assistant. Your task is to verify all submodules are properly configured and in sync.

## Submodule Architecture

This project uses nested submodules:

```
project/                    # Parent repo
├── .claude/                # Submodule → project-claude repo
│   ├── base/               # Submodule → claude-base
│   ├── <tech-stack>/       # Submodule(s) → nestjs, react, django, etc.
│   └── commands -> base/commands
```

---

## Step 1: Check Submodule Initialization

```bash
git submodule status --recursive
```

**Expected output:**
- Each line should start with a commit hash (not `-` which means uninitialized)
- No error messages

**If uninitialized:**
```bash
git submodule update --init --recursive
```

---

## Step 2: Check Submodule URLs

Verify `.gitmodules` URLs match actual remotes:

```bash
# Check parent .gitmodules
cat .gitmodules

# Check actual remote for .claude
cd .claude && git remote -v && cd ..

# Check .claude/.gitmodules
cat .claude/.gitmodules

# Check actual remotes for nested submodules
cd .claude/base && git remote -v && cd ../..
```

**Common issues:**
- URL mismatch between `.gitmodules` and actual remote
- Repo was renamed but `.gitmodules` wasn't updated

**Fix:** Update `.gitmodules` to match actual remote, then run:
```bash
git submodule sync --recursive
```

---

## Step 3: Check Branch Status

All submodules should be on `main` branch:

```bash
cd .claude/base && git branch --show-current && cd ../..
cd .claude && git branch --show-current && cd ..
```

**If detached HEAD:**
```bash
cd <submodule>
git checkout main
cd ..
```

---

## Step 4: Check for Uncommitted Changes

```bash
# Check nested submodules
cd .claude
for dir in */; do
  if [ -e "$dir/.git" ] && [ ! -L "$dir" ]; then
    echo "=== .claude/$dir ==="
    cd "$dir"
    git status --short
    cd ..
  fi
done
cd ..

# Check .claude itself
echo "=== .claude ==="
cd .claude && git status --short && cd ..

# Check parent
echo "=== Parent repo ==="
git status --short
```

**If uncommitted changes exist:** Run `/commit` to commit them properly.

---

## Step 5: Check Sync with Remote

```bash
# Fetch all remotes
git fetch --all --recurse-submodules

# Check if submodules are behind
cd .claude/base
git log HEAD..origin/main --oneline
cd ../..

cd .claude
git log HEAD..origin/main --oneline
cd ..
```

**If behind:** Run `/pull` to update.

---

## Step 6: Check Commands Symlink

```bash
ls -la .claude/commands
```

**Expected:** Symlink pointing to `base/commands`

**If broken or missing:**
```bash
rm -rf .claude/commands
ln -s base/commands .claude/commands
```

---

## Step 7: Report Results

Generate a health report:

```
=== Submodule Health Check ===

Initialization:
  ✓ All submodules initialized

URLs:
  ✓ .claude URL matches remote
  ✓ .claude/base URL matches remote

Branches:
  ✓ .claude/base on main
  ✓ .claude on main

Uncommitted Changes:
  ✓ No uncommitted changes in submodules
  OR
  ⚠ .claude/base has uncommitted changes

Sync Status:
  ✓ All submodules up to date
  OR
  ⚠ .claude/base is 3 commits behind origin/main

Commands:
  ✓ Symlink OK: commands -> base/commands

Overall: ✓ Healthy / ⚠ Issues Found
```

---

## Auto-Fix Mode

If `$ARGUMENTS` contains `--fix`, attempt to fix common issues:

1. **Uninitialized submodules:** `git submodule update --init --recursive`
2. **URL mismatch:** `git submodule sync --recursive`
3. **Detached HEAD:** `git checkout main` in each submodule
4. **Broken commands symlink:** Recreate symlink
5. **Behind remote:** `git pull origin main` in each submodule

**Note:** Auto-fix will NOT commit uncommitted changes. Use `/commit` for that.

---

## Common Issues and Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| Uninitialized | `-` prefix in status | `git submodule update --init --recursive` |
| Wrong URL | Clone fails | Fix `.gitmodules`, then `git submodule sync` |
| Detached HEAD | "(HEAD detached)" | `git checkout main` in submodule |
| Behind remote | "Your branch is behind" | `/pull` |
| Ahead of remote | "Your branch is ahead" | Need to push submodule |
| Uncommitted changes | "modified content" | `/commit` to commit properly |
| Broken symlink | Commands not found | Recreate `ln -s base/commands commands` |
