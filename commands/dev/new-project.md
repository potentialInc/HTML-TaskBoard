---
description: Create a complete new project with Claude config and boilerplate code
argument-hint: Project name (e.g., monkey, coaching-platform)
---

You are a full project setup assistant. This command creates a new project with the shared claude-workflow as `.claude` submodule.

## Step 0: Get Project Name

If `$ARGUMENTS` is provided, use it as the project name. Otherwise, ask:

```
What is the project name? (e.g., monkey, coaching-platform)
```

Store as `$PROJECT_NAME` (lowercase, hyphenated).

## Step 1: Gather Tech Stack

Use **AskUserQuestion** to ask for the complete tech stack:

### Backend Framework (Required)
- **NestJS** (Recommended) - TypeScript, TypeORM, JWT, Swagger
- **Django** - Python, DRF, SimpleJWT, drf-spectacular

Store as:
- `$BACKEND` = "nestjs" | "django"

### Frontend Framework(s) (Required, multiSelect: true)
- **React Web** - React 19, TailwindCSS 4, shadcn/ui
- **React Native** - NativeWind, React Navigation, Detox

At least one must be selected. Both can be selected.

Store as:
- `$FRONTENDS` = array of ["react", "react-native"] (at least one required)

### Dashboard(s) (Optional)

**How many web dashboards do you need?**

Use **AskUserQuestion** to gather dashboard requirements:

Question: "How many web dashboards do you need?"
Header: "Dashboards"
Options:
  1. "None" - No dashboard needed
  2. "1 Dashboard" - Single dashboard (e.g., Admin)
  3. "2 Dashboards" - Two dashboards (e.g., Admin + Coach)
  4. "3+ Dashboards" - Three or more dashboards

**If user selects 1+ dashboards**, ask follow-up for EACH dashboard:

Question: "What is the name/purpose of dashboard #N?" (User enters custom text: "Admin", "Coach", "Partner", "Vendor", etc.)
Header: "Dashboard #N"

Note: Dashboards use React Web (Next.js). Can be selected regardless of frontend choice (e.g., React Native mobile + React dashboard).

Store as:
- `$DASHBOARDS` = array of dashboard names (e.g., ["admin", "coach", "partner"])
- Each dashboard creates folder: `frontend-{name}-dashboard/`

## Step 2: Confirm Project Structure

Display the planned structure and ask for confirmation:

```
=== Project Setup Plan: $PROJECT_NAME ===

Claude Configuration:
  - Uses shared claude-workflow submodule
  - Contains: base, nestjs, django, react, react-native skills

Boilerplate Code:
  - backend/                  ← nestjs-starter-kit (if NestJS)
  - backend/                  ← django-starter-kit (if Django)
  - frontend/                 ← react-starter-kit (if React Web)
  - frontend-{name}-dashboard/ ← react-starter-kit (for each dashboard in $DASHBOARDS)
  - mobile/                   ← react-native-starter-kit (if React Native)

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

## Step 4: Add Claude Configuration

Add the shared claude-workflow as `.claude` submodule:

```bash
git submodule add --branch dev https://github.com/potentialInc/claude-workflow.git .claude
git submodule update --init --recursive
```

This provides:
- `base/` - shared commands and skills
- `nestjs/` - NestJS backend skills
- `django/` - Django REST Framework skills
- `react/` - React web skills
- `react-native/` - React Native mobile skills

## Step 5: Clone Boilerplate Repositories

### Boilerplate Mapping

| Selection | Repository | Folder |
|-----------|------------|--------|
| NestJS | `https://github.com/potentialInc/nestjs-starter-kit` | `backend/` |
| Django | `https://github.com/potentialInc/django-starter-kit` | `backend/` |
| React Web | `https://github.com/potentialInc/react-starter-kit` | `frontend/` |
| Dashboard (each) | `https://github.com/potentialInc/react-starter-kit` | `frontend-{name}-dashboard/` |
| React Native | `https://github.com/potentialInc/react-native-starter-kit` | `mobile/` |

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

  # Dashboard services (for each dashboard in $DASHBOARDS)
  # Port assignment: 5174 + index (e.g., admin=5174, coach=5175, partner=5176)
  frontend-{name}-dashboard:
    build:
      context: ./frontend-{name}-dashboard
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-{name}-dashboard
    restart: unless-stopped
    ports:
      - '{5174 + index}:5173'
    networks:
      - $PROJECT_NAME-network

networks:
  $PROJECT_NAME-network:
    driver: bridge
```

## Step 7: Create Project Documentation Structure

```bash
# Copy templates from claude-base
cp -r .claude/base/templates/claude-project/* .claude-project/

# Rename template files (remove .template suffix)
for f in .claude-project/docs/*.template.md; do
  mv "$f" "${f%.template.md}.md"
done

# Replace $PROJECT_NAME placeholder in all template files
find .claude-project -name "*.md" -exec sed -i '' "s/\$PROJECT_NAME/$PROJECT_NAME/g" {} \;

# Replace $BACKEND placeholder
find .claude-project -name "*.md" -exec sed -i '' "s/\$BACKEND/$BACKEND/g" {} \;

# Replace $FRONTENDS placeholder
find .claude-project -name "*.md" -exec sed -i '' "s/\$FRONTENDS/$FRONTENDS/g" {} \;

# Create dashboard-specific plan folders from generic template
for dashboard in $DASHBOARDS; do
  mkdir -p .claude-project/plans/frontend-${dashboard}-dashboard/
  cp .claude/base/templates/claude-project/plans/frontend-dashboard/* \
     .claude-project/plans/frontend-${dashboard}-dashboard/
done

# Remove generic template folder (already copied to specific dashboards)
rm -rf .claude-project/plans/frontend-dashboard/

# Append gitignore rules from template
cat .claude-project/gitignore.template >> .gitignore
rm .claude-project/gitignore.template
```

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
├── .claude/              # Claude Code configuration (shared)
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

- .claude/ submodule using shared claude-workflow
- Backend: $BACKEND
- Frontend: $FRONTENDS
- Docker orchestration configured

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## Step 10: Create GitHub Repo and Push

### 10.1 Create Main Project Repo on GitHub

```bash
gh repo create potentialInc/$PROJECT_NAME --private --description "$PROJECT_NAME - Full stack application"
```

### 10.2 Add Remote and Push main Branch

```bash
git remote add origin https://github.com/potentialInc/$PROJECT_NAME.git
git branch -M main
git push -u origin main
```

### 10.3 Create and Push dev Branch

```bash
git checkout -b dev
git push -u origin dev
```

### 10.4 Set dev as Default Branch (Optional)

```bash
# Set dev as the default branch for development workflow
gh repo edit potentialInc/$PROJECT_NAME --default-branch dev
```

### 10.5 Return to main Branch

```bash
git checkout main
```

## Step 11: Final Report

```
=== Project Setup Complete ===

$PROJECT_NAME/
├── .claude/                    # Shared claude-workflow
│   ├── base/                   → shared commands/skills
│   ├── nestjs/                 → NestJS backend skills
│   ├── django/                 → Django REST Framework skills
│   ├── react/                  → React web skills
│   └── react-native/           → React Native mobile skills
├── .claude-project/            # Project docs
│   ├── plans/
│   │   ├── backend/
│   │   ├── frontend/
│   │   ├── frontend-{name}-dashboard/  # For each dashboard
│   │   └── mobile/
│   ├── memory/
│   └── docs/
├── backend/                    # $BACKEND boilerplate
├── frontend/                   # React Web (if selected)
├── frontend-{name}-dashboard/  # For each dashboard in $DASHBOARDS
├── mobile/                     # React Native (if selected)
├── docker-compose.yml
├── .gitignore
└── README.md

GitHub Repo Created:
- https://github.com/potentialInc/$PROJECT_NAME (private)

Claude Configuration:
- Uses shared claude-workflow submodule
- https://github.com/potentialInc/claude-workflow

Branches Created:
- main (production-ready code)
- dev (active development)

Next Steps:
1. cd $PROJECT_NAME
2. docker-compose up -d
3. Begin development on dev branch!
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
rm -rf backend frontend frontend-dashboard mobile .claude .claude-project docker-compose.yml

# Clean up GitHub (if repo was created)
gh repo delete potentialInc/$PROJECT_NAME --yes
```
