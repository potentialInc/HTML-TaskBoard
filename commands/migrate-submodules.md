# migrate-submodules

Add the claude-nestjs and claude-react submodules to a project that only has claude-base.

## Usage

Run this command when you want to upgrade a project from the old single-submodule structure to the new 3-submodule architecture.

## Instructions

When the user runs this command, execute the following steps:

### Step 1: Check Current State

```bash
cd $CLAUDE_PROJECT_DIR/.claude
cat .gitmodules
ls -la
```

Verify:
- `base/` submodule exists
- `nestjs/` does NOT exist yet
- `react/` does NOT exist yet

### Step 2: Add Submodules

```bash
cd $CLAUDE_PROJECT_DIR/.claude

# Add NestJS submodule
git submodule add https://github.com/potentialInc/claude-nestjs.git nestjs

# Add React submodule
git submodule add https://github.com/potentialInc/claude-react.git react

# Initialize and update
git submodule update --init --recursive
```

### Step 3: Verify Structure

```bash
ls -la nestjs/
ls -la react/
cat .gitmodules
```

Expected output:
- `nestjs/` contains: agents/, skills/, docs/, hooks/
- `react/` contains: agents/, skills/, docs/
- `.gitmodules` shows 3 submodules: base, nestjs, react

### Step 4: Commit Changes

```bash
cd $CLAUDE_PROJECT_DIR/.claude
git add .gitmodules nestjs react
git commit -m "feat: Add claude-nestjs and claude-react submodules

- Add nestjs/ submodule for NestJS-specific Claude Code config
- Add react/ submodule for React-specific Claude Code config
- Structure now: base/ (shared) + nestjs/ + react/

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
git push origin main
```

### Step 5: Update Parent Project (if applicable)

If the `.claude/` directory is itself a submodule in the parent project:

```bash
cd $CLAUDE_PROJECT_DIR
git add .claude
git commit -m "chore: Update .claude submodule with framework submodules"
```

## After Migration

The project will have this structure:
```
.claude/
├── .gitmodules     # 3 submodules
├── base/           # Shared (agents, hooks, commands)
├── nestjs/         # NestJS-specific (backend-developer, backend-dev-guidelines)
├── react/          # React-specific (frontend-developer, frontend-dev-guidelines)
├── agents/         # Project-specific overrides
├── skills/         # Project-specific + skill-rules.json
├── hooks/          # Project-specific hooks
└── settings.json
```

## Benefits

- NestJS updates propagate to all projects using `claude-nestjs`
- React updates propagate to all projects using `claude-react`
- Can swap `react/` for `reactnative/` for mobile projects
- Clear separation of framework concerns
