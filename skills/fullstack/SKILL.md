---
skill_name: fullstack
applies_to_local_project_only: false
auto_trigger_regex: [fullstack, /fullstack, fullstack workflow, development pipeline, fullstack project]
tags: [orchestration, pipeline, fullstack, development-workflow]
related_skills: [project-doc-writer, new-project, pdf-to-prd]
---

# Fullstack Development Pipeline

Orchestrate complete full-stack development from project initialization to API integration.

---

## Quick Start

```
/fullstack {project-name}
```

---

## 7-Step Workflow

| Step | Phase | Description | Command/Skill |
|------|-------|-------------|---------------|
| 1 | Initialize | Create project with Claude config | `/new-project` |
| 2 | PRD Setup | Verify/convert PRD to markdown | `/pdf-to-prd` |
| 3 | Documentation | Update project docs from PRD | `project-doc-writer` |
| 4 | Database | Design database schema | Backend folder |
| 5 | API | Develop backend API | Backend folder |
| 6 | Screens | Implement frontend screens | **User Choice: Figma or HTML** |
| 7 | Integration | Connect frontend to backend | Frontend folder |

---

## Step 1: Initialize Project

**Goal:** Create project structure with Claude config and boilerplate

**Execute:**
```
/new-project {project-name}
```

**Outcome:**
- `.claude/` submodule configured
- `.claude-project/` documentation structure
- Boilerplate code (backend/, frontend/, etc.)
- `docker-compose.yml` generated

**Skip if:** Project already has `.claude/` and `.claude-project/`

**Reference:** [project-init.md](./project-init.md)

---

## Step 2: Verify PRD

**Goal:** Ensure PRD is in correct location and format

**Check PRD location:**
```bash
ls .claude-project/prd/
```

**If PRD is PDF and needs conversion:**
```
/pdf-to-prd /path/to/your-prd.pdf
```

**Expected Output:**
- `.claude-project/prd/{project}-prd.md` - Structured markdown PRD

**PRD Required Sections:**
1. Overview
2. Terminology
3. User Types
4. Project Structure
5. Page Architecture
6. Page List with Features

---

## Step 3: Update Project Documentation

**Goal:** Generate/update project docs from PRD

**Execute:**
```
/project-doc-writer
```

Or invoke skill directly: "Update project documentation from PRD"

**Documents Generated:**
- `.claude-project/docs/PROJECT_KNOWLEDGE.md`
- `.claude-project/docs/PROJECT_DATABASE.md`
- `.claude-project/docs/PROJECT_API.md`

**Reference:** `.claude/base/skills/project-doc-writer/SKILL.md`

---

## Step 4: Design Database

**Goal:** Design database schema and create migrations

**Working Directory:** `backend/`

**Pre-requisites:**
- Read `.claude-project/docs/PROJECT_DATABASE.md`
- Check `.claude-project/docs/PROJECT_KNOWLEDGE.md` for entity relationships

**Actions:**
1. Create TypeORM entities in `backend/src/modules/`
2. Generate migrations: `npm run migration:generate -- --name=InitialSchema`
3. Run migrations: `npm run migration:run`

**Update Status:**
- `.claude-project/status/backend/API_IMPLEMENTATION_STATUS.md`

**Reference:** `.claude/nestjs/guides/database-patterns.md`

---

## Step 5: Develop API

**Goal:** Implement backend REST API endpoints

**Working Directory:** `backend/`

**Pre-requisites:**
- Read `.claude-project/docs/PROJECT_API.md`
- Database entities created (Step 4)

**Four-Layer Implementation per Feature:**
1. **Entity** - TypeORM entity (extends BaseEntity)
2. **Repository** - Data access layer (extends BaseRepository)
3. **Service** - Business logic (extends BaseService)
4. **Controller** - HTTP endpoints (extends BaseController)
5. **DTOs** - Request/response validation
6. **Module** - NestJS module registration

**Update Status:**
- `.claude-project/status/backend/API_IMPLEMENTATION_STATUS.md`

**Reference:** `.claude/nestjs/skills/api-development/SKILL.md`

---

## Step 6: Implement Screens

**Goal:** Build frontend UI screens

**Working Directory:** `frontend/` or `frontend-admin-dashboard/`

### IMPORTANT: User Choice Required

Before proceeding, **MUST ASK** the user using AskUserQuestion:

```
Question: "How would you like to implement the frontend screens?"
Header: "Screen Method"
Options:
  - label: "Figma to React"
    description: "Convert Figma designs directly. Requires Figma link."
  - label: "HTML to React"
    description: "Convert HTML prototypes. Save HTML files in html/ folder first."
MultiSelect: false
```

### Option A: Figma to React

**User provides:** Figma link (e.g., `https://www.figma.com/design/{file_key}/{name}?node-id={node-id}`)

**Process:**
1. Extract node ID from URL (convert `-` to `:`)
2. Use Figma MCP tools:
   - `mcp__figma__get_design_context` - Get design data
   - `mcp__figma__get_screenshot` - Visual reference
3. Convert to React components with pixel-perfect styling
4. Update screen status tracking

**Follow-up question:**
```
Question: "Please provide the Figma file URL with the screen you want to implement"
Header: "Figma Link"
(Text input)
```

**Reference:** `.claude/react/skills/converters/figma-to-react-converter.md`

### Option B: HTML to React

**User action required:** Save HTML files to `html/` folder in project root

**Process:**
1. Confirm with user: "Please save your HTML files to the html/ folder, then confirm when ready"
2. Read HTML files from `html/` folder
3. Identify reusable components and patterns
4. Convert to React components:
   - `class` -> `className`
   - Inline scripts -> React hooks
   - `<a href>` -> `<Link to>`
5. Integrate with routing
6. Update screen status tracking

**Reference:** `.claude/react/skills/converters/html-to-react-converter.md`

### Update Status:
- `.claude-project/status/frontend/SCREEN_IMPLEMENTATION_STATUS.md`
- `.claude-project/status/frontend-admin-dashboard/SCREEN_IMPLEMENTATION_STATUS.md`

---

## Step 7: Integrate API

**Goal:** Connect frontend screens to backend API

**Working Directory:** `frontend/` or `frontend-admin-dashboard/`

**Pre-requisites:**
- Read `.claude-project/docs/PROJECT_API.md`
- Screens implemented (Step 6)
- Backend running

**Process:**
1. Create service files in `frontend/app/services/`
2. Define TypeScript types in `frontend/app/types/`
3. Integrate API calls in components
4. Add loading/error/success states
5. Test API integration

**Update Status:**
- `.claude-project/status/frontend/API_INTEGRATION_STATUS.md`
- `.claude-project/docs/PROJECT_API_INTEGRATION.md`

**Reference:** `.claude/react/skills/api-integration/SKILL.md`

---

## Status Tracking

### Pipeline Status File
`.claude-project/status/{project}/PIPELINE_STATUS.md`

### Per-Component Status Files

| Component | Status File |
|-----------|-------------|
| Backend API | `.claude-project/status/backend/API_IMPLEMENTATION_STATUS.md` |
| Frontend Screens | `.claude-project/status/frontend/SCREEN_IMPLEMENTATION_STATUS.md` |
| Frontend API Integration | `.claude-project/status/frontend/API_INTEGRATION_STATUS.md` |
| Dashboard Screens | `.claude-project/status/frontend-admin-dashboard/SCREEN_IMPLEMENTATION_STATUS.md` |

---

## Iteration Support

After completing initial implementation, use iteration manager for improvements:

```
/fullstack --new-iteration
```

**Commands:**
- `--new-iteration` - Start new iteration cycle
- `--iterate <phase>` - Re-run specific phase
- `--sync-prd` - Compare implementation against PRD
- `--compare <iter1> <iter2>` - Compare iterations

**Reference:** [iteration-manager.md](./iteration-manager.md)

---

## Deployment

When ready to deploy:

```
/fullstack --ship
```

**Environments:**
1. Development (Dokploy)
2. Staging (Dokploy)
3. Production (AWS ECS)

**Reference:** [deployment.md](./deployment.md)

---

## Related Skills

| Skill | Purpose | Location |
|-------|---------|----------|
| project-doc-writer | Generate docs from PRD | `.claude/base/skills/project-doc-writer/` |
| api-development | NestJS API development | `.claude/nestjs/skills/api-development/` |
| api-integration | Frontend API integration | `.claude/react/skills/api-integration/` |
| figma-to-react-converter | Figma to React | `.claude/react/skills/converters/` |
| html-to-react-converter | HTML to React | `.claude/react/skills/converters/` |

---

## Commands Reference

| Command | Description |
|---------|-------------|
| `/new-project` | Initialize new project |
| `/pdf-to-prd` | Convert PDF PRD to markdown |
| `/project-doc-writer` | Update project documentation |
| `/fullstack` | Run fullstack pipeline |
| `/fullstack --new-iteration` | Start new iteration |
| `/fullstack --ship` | Deploy to environments |

---

**Skill Status**: COMPLETE
