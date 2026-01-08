---
description: Create a complete new project with Claude config and boilerplate code
argument-hint: Project name (e.g., monkey, coaching-platform)
---

You are a full project setup assistant. This command combines `/initialize-claude` and `/create-mono-repo` into a single streamlined workflow.

## Step 0: Get Project Name

If `$ARGUMENTS` is provided, use it as the project name. Otherwise, ask:

```
What is the project name? (e.g., monkey, coaching-platform)
```

Store as `$PROJECT_NAME` (lowercase, hyphenated).

## Step 1: Gather Tech Stack

Use **AskUserQuestion** to ask for the complete tech stack:

### Backend Framework
- **NestJS** (Recommended) - TypeScript, TypeORM, JWT, Swagger
- **Django** - Python, DRF, SimpleJWT, drf-spectacular
- **None** - No backend needed

### Frontend Framework(s) (multiSelect: true)
- **React Web** - React 19, TailwindCSS 4, shadcn/ui
- **React Native** - NativeWind, React Navigation, Detox
- **None** - No frontend needed

Store selections as:
- `$BACKEND` = "nestjs" | "django" | null
- `$FRONTENDS` = array of ["react", "react-native"] (can be empty)

## Step 2: Confirm Project Structure

Display the planned structure and ask for confirmation:

```
=== Project Setup Plan: $PROJECT_NAME ===

Claude Configuration:
  - Repo: potentialInc/$PROJECT_NAME-claude
  - Submodules: base, $BACKEND, $FRONTENDS

Boilerplate Code:
  - backend/     ← nestjs-starter-kit (if NestJS)
  - backend/     ← django_boilerplate (if Django)
  - frontend/    ← react-19-starter-kit (if React Web)
  - mobile/      ← native-starter (if React Native)

Project Documentation:
  - .claude-project/plans/
  - .claude-project/memory/
  - .claude-project/docs/

Proceed with this setup?
```

## Step 3: Create Project Directory

```bash
# Check if we're in an empty directory or need to create one
if [ "$(ls -A .)" ]; then
    mkdir $PROJECT_NAME
    cd $PROJECT_NAME
fi

# Initialize git if not already
git init 2>/dev/null || true
```

## Step 4: Set Up Claude Configuration

Execute the `/initialize-claude` workflow internally:

### 4.1 Create Config Repo on GitHub

```bash
gh repo create potentialInc/$PROJECT_NAME-claude --public --description "Claude Code configuration for $PROJECT_NAME"
git clone https://github.com/potentialInc/$PROJECT_NAME-claude.git /tmp/$PROJECT_NAME-claude
cd /tmp/$PROJECT_NAME-claude
```

### 4.2 Add Framework Submodules

```bash
# Always add base
git submodule add https://github.com/potentialInc/claude-base.git base
```

Based on `$BACKEND`:
- NestJS: `git submodule add https://github.com/potentialInc/claude-nestjs.git nestjs`
- Django: `git submodule add https://github.com/potentialInc/claude-django.git django`

Based on `$FRONTENDS`:
- React: `git submodule add https://github.com/potentialInc/claude-react.git react`
- React Native: `git submodule add https://github.com/potentialInc/claude-react-native.git react-native`

```bash
git submodule update --init --recursive
```

### 4.3 Create Config Structure

```bash
mkdir -p agents hooks skills
ln -s base/commands commands

cat > .gitignore << 'EOF'
settings.local.json
*.local.*
EOF

cat > settings.json << 'EOF'
{
  "hooks": {
    "UserPromptSubmit": [],
    "PostToolUse": [],
    "Stop": []
  },
  "mcpServers": {}
}
EOF

cat > skills/skill-rules.json << 'EOF'
{
  "version": "1.0",
  "skills": {
    "backend-dev-guidelines": {
      "type": "domain",
      "enforcement": "suggest",
      "priority": "high",
      "promptTriggers": {
        "keywords": ["api", "backend", "controller", "service", "entity", "repository"]
      }
    },
    "frontend-dev-guidelines": {
      "type": "domain",
      "enforcement": "suggest",
      "priority": "high",
      "promptTriggers": {
        "keywords": ["react", "component", "frontend", "ui", "tsx", "page"]
      }
    }
  }
}
EOF
```

### 4.4 Push Config Repo

```bash
git add -A
git commit -m "$(cat <<'EOF'
feat: Initialize Claude Code config with modular submodules

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
git push -u origin main
```

### 4.5 Add .claude Submodule to Project

```bash
cd $PROJECT_DIR
git submodule add https://github.com/potentialInc/$PROJECT_NAME-claude.git .claude
git submodule update --init --recursive
```

## Step 5: Clone Boilerplate Repositories

Execute the `/create-mono-repo` workflow internally (without interactive prompts):

### Boilerplate Mapping

| Selection | Repository | Folder |
|-----------|------------|--------|
| NestJS | `https://github.com/potentialInc/nestjs-starter-kit` | `backend/` |
| Django | `https://github.com/potentialInc/django_boilerplate` | `backend/` |
| React Web | `https://github.com/potentialInc/react-19-starter-kit` | `frontend/` |
| React Native | `https://github.com/potentialInc/native-starter` | `mobile/` |

### Clone Each Selected Repo

```bash
# For each selected boilerplate:
git clone --branch main --single-branch $REPO_URL $FOLDER

# Remove Docker files from subfolder
rm -f $FOLDER/Dockerfile* $FOLDER/docker-compose*.yml $FOLDER/.dockerignore

# Remove .git to maintain single history
rm -rf $FOLDER/.git $FOLDER/.gitmodules
```

## Step 6: Generate docker-compose.yml

Create root `docker-compose.yml` based on selected services:

```yaml
version: '3.8'

services:
  # Backend service (if selected)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-backend
    restart: unless-stopped
    ports:
      - '3000:3000'  # NestJS
      # - '8000:8000'  # Django
    environment:
      - NODE_ENV=${NODE_ENV:-development}
    networks:
      - $PROJECT_NAME-network

  # Frontend service (if selected)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-frontend
    restart: unless-stopped
    ports:
      - '5173:5173'
    networks:
      - $PROJECT_NAME-network

networks:
  $PROJECT_NAME-network:
    driver: bridge
```

## Step 7: Create Project Documentation Structure

```bash
mkdir -p .claude-project/plans
mkdir -p .claude-project/memory
mkdir -p .claude-project/docs

# Create initial PROJECT_KNOWLEDGE.md
cat > .claude-project/memory/PROJECT_KNOWLEDGE.md << 'EOF'
# Project Knowledge: $PROJECT_NAME

## Overview
[Brief description of what this project does]

## Tech Stack
- Backend: $BACKEND
- Frontend: $FRONTENDS

## Architecture
[High-level architecture notes]

## Key Decisions
[Important technical decisions and rationale]
EOF

# Create initial README
cat > README.md << 'EOF'
# $PROJECT_NAME

## Quick Start

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# Or if already cloned
git submodule update --init --recursive

# Start services
docker-compose up -d
```

## Project Structure

```
$PROJECT_NAME/
├── .claude/              # Claude Code configuration
├── .claude-project/      # Project documentation
├── backend/              # Backend API
├── frontend/             # Web frontend
├── mobile/               # Mobile app (if applicable)
└── docker-compose.yml    # Container orchestration
```
EOF
```

## Step 8: Create .gitignore

```bash
cat > .gitignore << 'EOF'
# Node
**/node_modules/
**/dist/
**/build/

# Python
**/__pycache__/
**/*.pyc
**/.venv/

# Environment
**/.env
**/.env.local
**/.env.*.local

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db
EOF
```

## Step 9: Initial Commit

```bash
git add -A
git commit -m "$(cat <<'EOF'
feat: Initial $PROJECT_NAME project setup

- .claude/ submodule for Claude Code configuration
- Backend: $BACKEND
- Frontend: $FRONTENDS
- Docker orchestration configured

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## Step 10: Final Report

```
=== Project Setup Complete ===

$PROJECT_NAME/
├── .claude/                    # Claude Code config
│   ├── base/                   → claude-base
│   ├── $BACKEND/               → claude-$BACKEND
│   └── ...
├── .claude-project/            # Project docs
│   ├── plans/
│   ├── memory/
│   └── docs/
├── backend/                    # $BACKEND boilerplate
├── frontend/                   # React Web (if selected)
├── mobile/                     # React Native (if selected)
├── docker-compose.yml
├── .gitignore
└── README.md

GitHub Repos Created:
- potentialInc/$PROJECT_NAME-claude (Claude config)

Next Steps:
1. Create main project repo: gh repo create potentialInc/$PROJECT_NAME --private
2. Push: git remote add origin <url> && git push -u origin main
3. Start services: docker-compose up -d
4. Begin development!
```

## Error Handling

| Error | Resolution |
|-------|------------|
| `gh` not authenticated | Run `gh auth login` |
| Repo already exists | Ask to use existing or choose new name |
| Clone failed | Check network and repo access |
| Directory not empty | Ask to proceed or choose new location |

## Rollback

If setup fails midway:

```bash
# Clean up local
rm -rf backend frontend mobile .claude .claude-project docker-compose.yml

# Clean up GitHub (if config repo was created)
gh repo delete potentialInc/$PROJECT_NAME-claude --yes
```
