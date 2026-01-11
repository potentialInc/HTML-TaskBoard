---
description: Run autonomous workflow loops (design-qa, e2e-tests, backend-qa, api-docs)
argument-hint: "<project> [--run | --phase <name> | --reset <phase>] [--run-all]"
---

# Fullstack Pipeline Orchestrator

A skill-chain orchestrator that runs the full development lifecycle from project setup to deployment.

## Quick Start

```bash
# Show pipeline status
/fullstack my-project

# Run next pending phase
/fullstack my-project --run

# Run specific phase
/fullstack my-project --phase backend

# Run all remaining phases
/fullstack my-project --run-all

# Reset a phase to pending
/fullstack my-project --reset database
```

---

## Pipeline Phases

| # | Phase | Skill | Tier | Prerequisites | Output |
|---|-------|-------|------|---------------|--------|
| 1 | init | project-init.md | base | - | .claude-project/, .claude/ |
| 2 | prd | convert-prd-to-knowledge.md | nestjs | init | PROJECT_KNOWLEDGE.md |
| 3 | database | database-schema-designer.md | nestjs | prd | Entities, migrations |
| 4 | backend | (composite skills) | nestjs | database | API endpoints |
| 5 | frontend | (user choice: design-scratch / figma / html) | react | prd | React screens |
| 6 | integrate | api-integration.md | react | backend, frontend | Connected UI |
| 7 | test | e2e-test-generator.md | stack | integrate | E2E test specs |
| 8 | qa | design-qa.md + Ralph | react | test | 95% pass rate |
| 9 | ship | deployment.md | base | qa | Live deployment |

### Tier Locations

| Tier | Path | Description |
|------|------|-------------|
| base | `.claude/base/skills/fullstack/` | Generic orchestration skills |
| nestjs | `.claude/nestjs/skills/` | NestJS backend skills |
| react | `.claude/react/skills/` | React frontend skills |
| stack | Auto-detected based on project | Framework-specific |

---

## Execution Instructions

### Step 1: Parse Arguments

```
project = $1 (e.g., "my-project")
action = --run | --phase <name> | --reset <name> | --run-all | (none = status)
```

If no project is provided, ask using **AskUserQuestion**:
```
What is the project name?
```

### Step 2: Locate or Create Status File

Status file path: `.claude-project/plans/{project}/PIPELINE_STATUS.md`

**If status file doesn't exist:**
1. Copy template from `.claude/base/templates/PIPELINE_STATUS.template.md`
2. Replace `{PROJECT_NAME}` with actual project name
3. Set all phases to `Pending` status

### Step 3: Action Handler

#### Action: (none) - Show Status

Read the status file and display:

```
Fullstack Pipeline - {project}
==============================

Phase       | Status      | Output
------------|-------------|------------------
init        | Complete    | .claude-project/
prd         | Complete    | PROJECT_KNOWLEDGE.md
database    | In Progress | migrations/
backend     | Pending     | -
frontend    | Pending     | -
integrate   | Pending     | -
test        | Pending     | -
qa          | Pending     | -
ship        | Pending     | -

Next: database (run with --run)
```

#### Action: --run - Execute Next Pending Phase

1. Read status file
2. Find first phase where:
   - Status = `Pending` OR `Failed`
   - All prerequisites have Status = `Complete`
3. If no eligible phase found, report "Pipeline complete" or "Blocked"
4. Execute the phase (see Step 4)

#### Action: --phase <name> - Execute Specific Phase

1. Validate phase name exists
2. Check prerequisites are complete
3. If prerequisites not met, show error with missing dependencies
4. Execute the phase (see Step 4)

#### Action: --run-all - Execute All Remaining Phases

1. Loop while there are pending phases:
   - Run next eligible phase
   - If phase fails, stop and report
   - If all phases complete, report success

#### Action: --reset <name> - Reset Phase to Pending

1. Update status file: set phase status to `Pending`
2. Clear the Notes column
3. Optionally ask to reset dependent phases too

### Step 4: Execute a Phase

For each phase execution:

**4.1 Update Status**
```
Set phase status = In Progress (icon)
```

**4.2 Load Skill (Tier-Aware)**

Look up the skill path based on the phase-to-tier mapping:

```
Phase → Tier → Skill Path
─────────────────────────────────────────────────────────────────
init      → base   → .claude/base/skills/fullstack/project-init.md
prd       → nestjs → .claude/nestjs/skills/convert-prd-to-knowledge.md
database  → nestjs → .claude/nestjs/skills/database-schema-designer.md
backend   → nestjs → (use architecture-overview.md + services-and-repositories.md)
frontend  → react  → (multi-path - see "Frontend Phase: Multi-Path Selection")
                     ├─ design-scratch → /prd-to-design-prompts (command)
                     ├─ figma         → .claude/react/skills/convert-figma-to-react.md
                     └─ html          → .claude/react/skills/convert-html-to-react.md
integrate → react  → .claude/react/guides/api-integration.md
test      → stack  → .claude/{detected-stack}/skills/e2e-test-generator.md
qa        → react  → .claude/react/skills/design-qa-patterns.md (+ /ralph for iteration)
ship      → base   → .claude/base/skills/fullstack/deployment.md
```

Read the skill file from its correct tier location and follow its instructions.

**4.3 Execute Skill Instructions**

Each skill file contains:
- Context (what phase expects)
- Instructions (what to do)
- Completion Criteria (how to verify success)
- On Success / On Failure actions

**4.4 Update Status Based on Result**

On Success:
```
Set phase status = Complete (white_check_mark)
Update Output column with deliverables
Add to Execution Log
```

On Failure:
```
Set phase status = Failed (x)
Add error details to Notes column
Add to Execution Log
STOP execution (do not continue to next phase)
```

---

## Frontend Phase: Multi-Path Selection

The frontend phase supports three implementation paths. The agent MUST ask the user to choose.

### Step F1: Present Options

Use **AskUserQuestion** with:

```
Question: "How would you like to implement the frontend screens?"
Header: "Frontend Path"
Options:
  1. "Design from scratch" - Generate design prompts from PRD, create designs externally, then implement
  2. "Convert from Figma" - Implement from existing Figma designs using MCP tools
  3. "Convert from HTML" - Convert existing HTML/Tailwind templates to React
```

### Step F2: Execute Selected Path

#### Path A: Design from Scratch

1. Run `/prd-to-design-prompts {prd-path} --tool generic`
2. Update status: `frontend` | `Blocked` | `Awaiting external designs`
3. Report output location and next steps:
   - Prompts saved to `.claude-project/design-prompts/`
   - User should create designs using AI tools (Aura, v0, Gemini, etc.)
   - Re-run `/fullstack {project} --phase frontend` when designs are ready
4. **STOP** - User must create designs externally
5. On re-run: ask if designs are ready → proceed to Figma or HTML conversion

#### Path B: Convert from Figma

1. Ask for Figma URL(s) or use PROJECT_KNOWLEDGE.md figma links
2. Load skill: `.claude/react/skills/convert-figma-to-react.md`
3. Execute per skill instructions
4. Update status on completion

#### Path C: Convert from HTML

1. Ask for HTML file path(s)
2. Load skill: `.claude/react/skills/convert-html-to-react.md`
3. Execute per skill instructions
4. Update status on completion

### Step F3: Status Updates

| Path | On Success Status | Notes |
|------|------------------|-------|
| Design from scratch (prompts) | Blocked | "Awaiting external designs - prompts at {path}" |
| Design from scratch (implement) | Complete | "Implemented from {source}" |
| Figma | Complete | "Converted from Figma" |
| HTML | Complete | "Converted from HTML templates" |

---

## Status File Format

```markdown
# Fullstack Pipeline Status - {PROJECT_NAME}

## Progress

| Phase | Skill | Tier | Status | Prerequisites | Output | Notes |
|-------|-------|------|--------|---------------|--------|-------|
| init | project-init.md | base | :clipboard: | - | - | - |
| prd | convert-prd-to-knowledge.md | nestjs | :clipboard: | init | - | - |
| database | database-schema-designer.md | nestjs | :clipboard: | prd | - | - |
| backend | (composite) | nestjs | :clipboard: | database | - | - |
| frontend | convert-figma-to-react.md | react | :clipboard: | prd | - | - |
| integrate | api-integration.md | react | :clipboard: | backend, frontend | - | - |
| test | e2e-test-generator.md | stack | :clipboard: | integrate | - | - |
| qa | design-qa.md | react | :clipboard: | test | - | - |
| ship | deployment.md | base | :clipboard: | qa | - | - |

## Execution Log

| Date | Phase | Duration | Result | Notes |
|------|-------|----------|--------|-------|

## Configuration

```yaml
project: {PROJECT_NAME}
created: {DATE}
last_run: null
tech_stack: nestjs+react  # Auto-detected or user-specified
```
```

---

## Prerequisites Validation

Before executing a phase, check prerequisites:

```
For phase = "backend":
  prerequisites = ["database"]

  For each prerequisite:
    Check status in Progress table
    If status != Complete:
      FAIL with message: "Cannot run 'backend': prerequisite 'database' is not complete"
```

---

## Hybrid Mode: Ralph for Item-Level Work

Some phases delegate to Ralph for item-level iteration:

| Phase | Items | Delegate To |
|-------|-------|-------------|
| frontend | 48 screens | `/ralph design-qa project --incremental` |
| test | 29 tests | E2E test generation skill |
| qa | Multiple checks | `/ralph e2e-tests`, `/ralph design-qa` |

The skill file determines when to delegate to Ralph.

Example from `qa-runner.md`:
```markdown
## Execution

1. Run: /ralph e2e-tests {project} --incremental
2. Run: /ralph design-qa {project} --incremental
3. Calculate combined pass rate from status files
4. If pass_rate >= 95%: mark qa as Complete
5. Else: mark qa as Failed, list failing items
```

---

## Error Handling

### Phase Fails

1. Mark phase status = Failed
2. Add error to Notes column
3. Add to Execution Log
4. **STOP** - do not continue to next phase
5. Report failure with suggested fix

### Missing Skill File

1. Report error: "Skill file not found: .claude/base/skills/fullstack/{phase}.md"
2. Suggest creating the skill or checking path

### Missing Prerequisites

1. Report which prerequisites are missing
2. Suggest running `--phase <prerequisite>` first

### Status File Corruption

1. If status file can't be parsed, offer to reset it
2. Never lose user's progress without confirmation

---

## Skill File Structure

Skills are located in their appropriate tier (see Tier Locations above).

**Base tier skills** (in `.claude/base/skills/fullstack/`) are orchestration-specific:
- `project-init.md` - Generic project initialization
- `deployment.md` - Generic deployment (Dokploy/AWS)

**Framework tier skills** (in `.claude/{nestjs|react}/skills/`) are comprehensive implementations:
- Already exist and are well-maintained
- Used by both standalone invocation AND /fullstack pipeline
- Single source of truth for each capability

Skill files follow this pattern:

```markdown
---
name: {phase-name}
phase: {phase-number}
prerequisites: [{list}]
---

# {Phase Name} Skill

## Context
- Project: {from status file}
- Previous phase output: {what to expect}
- Expected deliverables: {what this phase produces}

## Instructions

1. Step one...
2. Step two...
3. Step three...

## Completion Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## On Success

Update PIPELINE_STATUS.md:
- Status: :white_check_mark:
- Output: {deliverables}

## On Failure

Update PIPELINE_STATUS.md:
- Status: :x:
- Notes: {error details}
```

---

## Related Commands

- `/new-project` - Create new project with Claude config (used by init phase)
- `/pdf-to-prd` - Convert PRD PDF to markdown (used by prd phase)
- `/ralph` - Run item-level workflows (used by qa phase)

---

## Examples

### Start New Project

```bash
/fullstack my-app --run
# Runs init phase, creates .claude-project/
```

### Resume After Failure

```bash
/fullstack my-app
# Shows: database phase failed
# Notes: "Migration error - duplicate column"

# Fix the issue manually, then:
/fullstack my-app --phase database
# Retries the database phase
```

### Skip to Specific Phase

```bash
# Prerequisites must be complete
/fullstack my-app --phase frontend
# Error if database/backend not done
```

### Full Automated Run

```bash
/fullstack my-app --run-all
# Runs all phases until complete or failure
```
