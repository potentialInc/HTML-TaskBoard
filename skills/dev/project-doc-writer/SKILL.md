---
skill_name: project-doc-writer
applies_to_local_project_only: false
auto_trigger_regex: [project documentation, update docs, generate docs, PROJECT_KNOWLEDGE, PROJECT_DATABASE, PROJECT_API, strategic plan, claude-project, prd to docs, sync documentation, documentation writer]
tags: [documentation, prd, project-knowledge, project-database, project-api, plans, claude-project]
related_skills: [generate-prd, pdf-to-prd, init-workspace]
---

# Project Documentation Writer

Generate and maintain project documentation in `.claude-project/` folder based on PRD files.

---

## Purpose

This skill helps you:

1. **Read PRD files** from `.claude-project/prd/`
2. **Generate documentation** for `docs/` and `plans/` folders
3. **Keep docs in sync** with PRD changes
4. **Create strategic plans** from project requirements

---

## Quick Start

### Generate All Documentation

```
/project-doc-writer
```

### Update Specific Document

```
/project-doc-writer --doc=PROJECT_KNOWLEDGE
/project-doc-writer --doc=PROJECT_DATABASE
/project-doc-writer --doc=PROJECT_API
```

### Incremental Update Mode

```
/project-doc-writer --incremental
```

---

## Workflow Overview

### Step 1: Locate PRD

- Check `.claude-project/prd/` for PRD files
- Support formats: PDF, Markdown (.md)
- If no PRD found: Use AskUserQuestion to request PRD path or upload

### Step 2: Parse PRD Content

Extract key sections:

- Overview/Description
- User Types & Permissions
- Database Schema (explicit or inferred)
- API Endpoints (explicit or inferred from features)
- Features & Pages
- Tech Stack

### Step 3: Choose Update Mode

- **Full Update** (default): Regenerate all docs with preview
- **Incremental**: Ask questions for each section before updating

### Step 4: Generate Documents

| Document | Target Path |
|----------|-------------|
| PROJECT_KNOWLEDGE.md | `.claude-project/docs/` |
| PROJECT_DATABASE.md | `.claude-project/docs/` |
| PROJECT_API.md | `.claude-project/docs/` |
| Strategic plans | `.claude-project/plans/` |

### Step 5: Validate & Report

- Cross-reference check (entities, endpoints, features)
- Summary of changes made
- Warnings for missing mappings

---

## Target Directory Structure

```
.claude-project/
├── prd/                    # Source: PRD files
│   └── [ProjectName]_PRD.pdf
├── docs/                   # Target: Generated docs
│   ├── PROJECT_KNOWLEDGE.md
│   ├── PROJECT_DATABASE.md
│   └── PROJECT_API.md
├── plans/                  # Target: Strategic plans
│   └── [feature]_PLAN.md
└── memory/                 # Target: Updates
    ├── DECISIONS.md
    └── LEARNINGS.md
```

---

## Document Generation Rules

### PROJECT_KNOWLEDGE.md

**Source PRD Sections**:

- Part 1: Basic Information
- Project Type & Tech Stack
- User Types & Permissions
- System Modules

**Generated Sections**:

- Overview (name, description, goals)
- Tech Stack
- User Roles & Permissions
- Core Features (grouped by user type)
- Page Breakdown (Public, User, Admin)
- Categories/Enums

### PROJECT_DATABASE.md

**Source PRD Sections**:

- Data entities mentioned in features
- User types → User entity
- Features → Related entities

**Generated Sections**:

- Entity Overview Table
- Entity Details (columns, types, constraints)
- Relationships (1:N, M:N)
- Indexes
- Enums
- ER Diagram (ASCII art)

### PROJECT_API.md

**Source PRD Sections**:

- Page features → API endpoints
- User permissions → Auth requirements
- Data operations → CRUD endpoints

**Generated Sections**:

- Authentication Endpoints
- Resource Endpoints (per entity)
- Admin Endpoints
- Common Responses
- Pagination Pattern
- Error Format

---

## Update Modes

### Full Update Mode (Default)

Regenerates all documentation from PRD:

1. Parse PRD completely
2. Show preview of changes
3. Ask for confirmation
4. Generate fresh documents
5. Report summary

### Incremental Update Mode

Interactive section-by-section updates:

1. Show current section content
2. Show proposed new content from PRD
3. Ask: "Update this section? [Yes/No/Edit manually]"
4. Apply changes based on choice
5. Continue to next section

---

## PRD Handling

### Supported Formats

- **PDF**: Claude can read PDF directly
- **Markdown**: Direct text parsing
- **Multiple PRDs**: Ask user which to use

### Missing PRD Flow

If no PRD found in `.claude-project/prd/`:

```
No PRD found in .claude-project/prd/

Please choose an option:
1. Provide path to existing PRD file
2. Upload/paste PRD content
3. Generate docs from existing codebase instead
4. Cancel
```

Uses AskUserQuestion tool to gather PRD source.

---

## Command Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| (none) | Full update all docs | `/project-doc-writer` |
| `--doc=NAME` | Update specific doc | `--doc=PROJECT_API` |
| `--incremental` | Interactive mode | `--incremental` |
| `--prd=PATH` | Specify PRD path | `--prd=/path/to/prd.pdf` |
| `--plan=NAME` | Generate specific plan | `--plan=authentication` |

---

## Example Session

```
User: /project-doc-writer

Claude: Checking .claude-project/prd/ for PRD files...
Found: MyProject_PRD_v2.pdf

Parsing PRD content...
Extracted:
- Project: My Project
- User Types: 3 (Guest, User, Admin)
- Database Entities: 7
- API Endpoints: ~35

Preview:
- PROJECT_KNOWLEDGE.md: 12 sections to generate
- PROJECT_DATABASE.md: 7 entities to document
- PROJECT_API.md: 35 endpoints to document

Proceed with full update? [Y/n]

User: Y

Generating documentation...
✓ PROJECT_KNOWLEDGE.md (320 lines)
✓ PROJECT_DATABASE.md (280 lines)
✓ PROJECT_API.md (450 lines)

Files updated in .claude-project/docs/
```

---

## Validation Checks

After generation, the skill validates:

- [ ] All user types from PRD are documented
- [ ] All entities have database definitions
- [ ] All features have related API endpoints
- [ ] No orphan references between docs
- [ ] Consistent terminology across all docs

---

## Related Skills

- **generate-prd**: Create PRD from client input
- **pdf-to-prd**: Convert PDF PRD to markdown format
- **init-workspace**: Initialize .claude-project folder structure

---

## Reference Files

For detailed instructions, see:

- [prompts/analyze-prd.md](prompts/analyze-prd.md) - PRD parsing rules
- [prompts/generate-knowledge.md](prompts/generate-knowledge.md) - Knowledge doc generation
- [prompts/generate-database.md](prompts/generate-database.md) - Database doc generation
- [prompts/generate-api.md](prompts/generate-api.md) - API doc generation
- [prompts/update-incremental.md](prompts/update-incremental.md) - Incremental update flow

### Document Templates (shared)

Templates are located in `.claude/base/templates/claude-project/docs/`:

- `PROJECT_KNOWLEDGE.template.md`
- `PROJECT_DATABASE.template.md`
- `PROJECT_API.template.md`

---

**Skill Status**: COMPLETE
**Line Count**: ~300 (following 500-line rule)
**Progressive Disclosure**: Details in prompts/ and templates/ folders
