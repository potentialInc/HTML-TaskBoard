# migrate-submodules

Add framework-specific Claude Code submodules to a project that has claude-base.

## Usage

```
/migrate-submodules [options]
```

### Options

| Flag | Submodule | Description |
|------|-----------|-------------|
| `--nestjs` | `claude-nestjs` | NestJS backend (controllers, services, DTOs, Swagger) |
| `--django` | `claude-django` | Django REST Framework backend (models, serializers, ViewSets) |
| `--react` | `claude-react` | React web frontend (components, state, Playwright tests) |
| `--react-native` | `claude-react-native` | React Native mobile (NativeWind, React Navigation, Detox) |

### Common Combinations

| Project Type | Command |
|--------------|---------|
| NestJS + React Web | `/migrate-submodules --nestjs --react` |
| NestJS + React Native | `/migrate-submodules --nestjs --react-native` |
| Django + React Web | `/migrate-submodules --django --react` |

## Instructions

When the user runs this command with flags, execute the following steps:

### Step 1: Check Current State

```bash
cd $CLAUDE_PROJECT_DIR/.claude
cat .gitmodules
ls -la
```

Verify:
- `base/` submodule exists
- Requested submodules do NOT exist yet

### Step 2: Add Requested Submodules

Based on the flags provided, add the appropriate submodules:

**For `--nestjs`:**
```bash
cd $CLAUDE_PROJECT_DIR/.claude
git submodule add https://github.com/potentialInc/claude-nestjs.git nestjs
```

**For `--django`:**
```bash
cd $CLAUDE_PROJECT_DIR/.claude
git submodule add https://github.com/potentialInc/claude-django.git django
```

**For `--react`:**
```bash
cd $CLAUDE_PROJECT_DIR/.claude
git submodule add https://github.com/potentialInc/claude-react.git react
```

**For `--react-native`:**
```bash
cd $CLAUDE_PROJECT_DIR/.claude
git submodule add https://github.com/potentialInc/claude-react-native.git react-native
```

**After adding submodules:**
```bash
git submodule update --init --recursive
```

### Step 3: Verify Structure

```bash
ls -la */  # List all submodule directories
cat .gitmodules
```

Expected contents per submodule:
- `nestjs/`: agents/, skills/, docs/, hooks/
- `django/`: agents/, skills/, hooks/
- `react/`: agents/, skills/, docs/
- `react-native/`: agents/, skills/

### Step 4: Commit Changes

Adjust the commit message based on which submodules were added:

```bash
cd $CLAUDE_PROJECT_DIR/.claude
git add .gitmodules <added-submodules>
git commit -m "feat: Add framework-specific Claude Code submodules

- Added submodules for framework-specific configuration
- Structure now: base/ (shared) + framework-specific/

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

## Project Structure Examples

### NestJS + React Web
```
.claude/
├── .gitmodules     # 3 submodules
├── base/           # Shared (generic agents, hooks, commands)
├── nestjs/         # NestJS backend (backend-developer, NestJS patterns)
├── react/          # React frontend (frontend-developer, React patterns)
├── agents/         # Project-specific overrides
├── skills/         # Project-specific + skill-rules.json
└── settings.json
```

### NestJS + React Native
```
.claude/
├── .gitmodules     # 3 submodules
├── base/           # Shared
├── nestjs/         # NestJS backend
├── react-native/   # React Native (NativeWind, React Navigation, Detox)
├── agents/         # Project-specific
└── settings.json
```

### Django + React Web
```
.claude/
├── .gitmodules     # 3 submodules
├── base/           # Shared
├── django/         # Django backend (DRF, SimpleJWT, pytest-django)
├── react/          # React frontend
├── agents/         # Project-specific
└── settings.json
```

## Submodule Repository URLs

| Submodule | Repository |
|-----------|------------|
| base | https://github.com/potentialInc/claude-base |
| nestjs | https://github.com/potentialInc/claude-nestjs |
| django | https://github.com/potentialInc/claude-django |
| react | https://github.com/potentialInc/claude-react |
| react-native | https://github.com/potentialInc/claude-react-native |

## Benefits

- Backend updates (NestJS/Django) propagate to all projects using that framework
- Frontend updates (React/React Native) propagate to all projects using that framework
- Clear separation of framework concerns
- Mix and match: NestJS+React, NestJS+RN, Django+React, etc.
