# {PROJECT_NAME} - Project Configuration

## Overview

**Project**: {PROJECT_NAME}
**Type**: {PROJECT_TYPE}
**Status**: Development
**Version**: 1.0

{PROJECT_DESCRIPTION}

**Key Differentiator**: {KEY_DIFFERENTIATOR}

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | {BACKEND} |
| Frontend | {FRONTEND} |
| Admin Dashboard | {ADMIN_DASHBOARD} |
| Database | {DATABASE} |
| ORM | {ORM} |
| Authentication | {AUTH} |
| Deployment | {DEPLOYMENT} |

---

## Project Structure

```
{PROJECT_NAME}/
├── backend/                      # {BACKEND} API
│   ├── src/
│   │   ├── modules/             # Feature modules
│   │   ├── shared/              # Shared utilities
│   │   └── database/            # Migrations, seeds
│   └── test/                    # E2E tests
├── frontend/                    # {FRONTEND} user application
├── frontend-admin-dashboard/    # Admin dashboard
├── .claude/                     # Claude Code configuration
├── .claude-project/             # Project documentation
└── docker-compose.yml           # Docker configuration
```

---

## Key Documentation

| Document | Path | Description |
|----------|------|-------------|
| Project Knowledge | [.claude-project/docs/PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md) | Core project info, features, roles |
| API Documentation | [.claude-project/docs/PROJECT_API.md](.claude-project/docs/PROJECT_API.md) | API endpoints and specifications |
| Database Schema | [.claude-project/docs/PROJECT_DATABASE.md](.claude-project/docs/PROJECT_DATABASE.md) | Entity definitions and relationships |
| API Integration | [.claude-project/docs/PROJECT_API_INTEGRATION.md](.claude-project/docs/PROJECT_API_INTEGRATION.md) | Frontend-backend integration |

### Status Tracking

| Status File | Path |
|-------------|------|
| Backend API Status | [.claude-project/status/backend/API_IMPLEMENTATION_STATUS.md](.claude-project/status/backend/API_IMPLEMENTATION_STATUS.md) |
| Frontend Screens | [.claude-project/status/frontend/SCREEN_IMPLEMENTATION_STATUS.md](.claude-project/status/frontend/SCREEN_IMPLEMENTATION_STATUS.md) |
| Frontend API Integration | [.claude-project/status/frontend/API_INTEGRATION_STATUS.md](.claude-project/status/frontend/API_INTEGRATION_STATUS.md) |

### Memory (Persistent Context)

| File | Purpose |
|------|---------|
| [DECISIONS.md](.claude-project/memory/DECISIONS.md) | Architecture decisions log |
| [LEARNINGS.md](.claude-project/memory/LEARNINGS.md) | Patterns and insights |
| [PREFERENCES.md](.claude-project/memory/PREFERENCES.md) | Coding style preferences |

---

## User Roles

| Role | Description | Key Permissions |
|------|-------------|-----------------|
| Guest | Unauthenticated visitor | View public pages, Login, Signup |
| User | Registered user | {USER_PERMISSIONS} |
| Admin | Platform administrator | {ADMIN_PERMISSIONS} |

---

## Core Enums

| Enum | Values |
|------|--------|
| RoleEnum | `user`, `admin` |
| UserStatusEnum | `active`, `suspended` |

---

## Development Conventions

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useAuth.ts` |
| Utilities | camelCase | `formatDate.ts` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Types/Interfaces | PascalCase | `UserResponse` |

### Import Order

1. External packages
2. Internal aliases (@/)
3. Relative imports
4. Styles

### TypeScript Guidelines

- Strict mode enabled
- Prefer `interface` over `type` for objects
- Use `unknown` over `any`
- Explicit types over type inference

---

## Available Commands

| Command | Description |
|---------|-------------|
| `/commit` | Smart git commit with branch and PR management |
| `/pull` | Pull latest changes for all submodules |
| `/ralph <workflow> <project>` | Run autonomous QA/testing loops |
| `/new-project <name>` | Create complete new project with Claude config |

---

## Enabled MCP Servers

- **sequential-thinking** - Step-by-step reasoning
- **playwright** - Browser automation and E2E testing

---

## Design System

### Primary Color
- **{PRIMARY_COLOR_NAME}**: {PRIMARY_COLOR_HEX}

---

## API Base URLs

| Environment | URL |
|-------------|-----|
| Development | `http://localhost:3000/api` |
| Staging | `https://staging.{PROJECT_DOMAIN}/api` |
| Production | `https://api.{PROJECT_DOMAIN}/api` |

---

## Quick Reference

### Authentication Flow
- JWT Bearer tokens for API authentication
- Access Token: 1 hour expiry
- Refresh Token: 7 days expiry

### Database Design Decisions
- **UUID primary keys** for security and distributed system support
- **Soft deletes** with `deleted_at` timestamp

---

*Last Updated: {DATE}*
