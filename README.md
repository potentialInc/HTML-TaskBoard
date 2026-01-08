# claude-base

Shared Claude Code configuration - the base layer for all projects.

## Overview

This repository contains framework-agnostic configurations, commands, agents, and templates that are shared across all projects using the Claude Code architecture.

## Structure

```
claude-base/
├── agents/           # Reusable agent definitions
├── commands/         # Slash commands (/new-project, /commit, etc.)
├── docs/             # Generic documentation
├── hooks/            # Git and Claude hooks
├── skills/           # Shared skills
└── templates/        # Project templates
    └── claude-project/  # .claude-project folder templates
```

## Commands

| Command | Description |
|---------|-------------|
| `/new-project` | Create a complete new project with Claude config and boilerplate |
| `/initialize-claude` | Initialize Claude Code configuration only |
| `/create-mono-repo` | Clone boilerplate repos into mono-repo structure |
| `/commit` | Smart git commit with branch management |
| `/pull` | Pull latest changes for all submodules |

## Templates

The `templates/claude-project/` folder contains templates for project documentation:

```
templates/claude-project/
├── docs/
│   ├── PROJECT_KNOWLEDGE.template.md
│   ├── PROJECT_API.template.md
│   ├── PROJECT_DATABASE.template.md
│   └── PROJECT_API_INTEGRATION.template.md
├── plans/
│   └── temp/.gitkeep
├── memory/
│   └── .gitkeep
├── prd/
│   └── .gitkeep
├── secrets/
│   └── .gitkeep
└── gitignore.template
```

### Template Variables

Templates use these placeholders that get replaced during project creation:

| Variable | Description |
|----------|-------------|
| `$PROJECT_NAME` | Project name (lowercase, hyphenated) |
| `$BACKEND` | Selected backend framework |
| `$FRONTENDS` | Selected frontend frameworks |

## Usage

### As a Submodule

This repo is used as a submodule in project-specific Claude config repos:

```bash
# In your project's .claude repo
git submodule add https://github.com/potentialInc/claude-base.git base
```

### Architecture

```
project/
├── .claude/                    # Project-specific Claude config (submodule)
│   ├── base/                   → claude-base (this repo)
│   ├── nestjs/                 → claude-nestjs (if NestJS)
│   ├── react/                  → claude-react (if React)
│   └── settings.json
└── .claude-project/            # Project documentation (from templates)
    ├── docs/
    ├── plans/
    ├── memory/
    ├── prd/
    └── secrets/
```

## Related Repositories

### Claude Config Repos (Submodules)

| Repo | Purpose |
|------|---------|
| [claude-base](https://github.com/potentialInc/claude-base) | Shared base (this repo) |
| [claude-nestjs](https://github.com/potentialInc/claude-nestjs) | NestJS patterns |
| [claude-django](https://github.com/potentialInc/claude-django) | Django patterns |
| [claude-react](https://github.com/potentialInc/claude-react) | React patterns |
| [claude-react-native](https://github.com/potentialInc/claude-react-native) | React Native patterns |

### Boilerplate Repos (Cloned by /create-mono-repo)

| Repo | Purpose |
|------|---------|
| [nestjs-starter-kit](https://github.com/potentialInc/nestjs-starter-kit) | NestJS backend |
| [django-starter-kit](https://github.com/potentialInc/django-starter-kit) | Django backend |
| [react-starter-kit](https://github.com/potentialInc/react-starter-kit) | React frontend |
| [react-native-starter-kit](https://github.com/potentialInc/react-native-starter-kit) | React Native mobile |

## Contributing

1. Make changes to this repo
2. Commit and push
3. Update submodule reference in dependent projects:
   ```bash
   cd .claude/base
   git pull origin main
   cd ../..
   git add .claude
   git commit -m "chore: Update claude-base submodule"
   ```
