# Full-Stack Development Orchestrator Plan (v2)

## Design Principles

1. **Ralph-Centric**: Every phase runs as a Ralph autonomous loop
2. **Parallel Execution**: Frontend + Database phases run concurrently
3. **Zero Human Checkpoints**: Fully autonomous with status file gates
4. **Deployment**: Dokploy for dev/staging, AWS for production

---

## Architecture: Ralph-Powered Pipeline

### Core Concept

Instead of a single `/fullstack` command, we create **Ralph workflows for each phase**. The master command orchestrates which Ralph workflows to run and in what order.

```
┌─────────────────────────────────────────────────────────────────┐
│                  /fullstack (Orchestrator)                       │
│  Reads PIPELINE_STATUS.md → Determines next phases → Launches   │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Ralph Workflow Engine                         │
│  /ralph <workflow> <project> --completion-promise <PHASE_DONE>  │
└───────────────────────────────┬─────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        ▼                       ▼                       ▼
   ┌─────────┐            ┌─────────┐            ┌─────────┐
   │ Phase N │            │ Phase N │            │ Phase N │
   │ Status  │            │  Skill  │            │ Output  │
   │  File   │◄──────────►│  Logic  │───────────►│  Files  │
   └─────────┘            └─────────┘            └─────────┘
```

---

## Ralph Workflow Registry (Extended)

### Current Workflows
```yaml
workflows:
  design-qa:
    stack: react
    skill: design-qa.md
    status_file: SCREEN_IMPLEMENTATION_STATUS.md

  e2e-tests:
    stack: react
    skill: e2e-test-generator.md
    status_file: E2E_TEST_STATUS.md
```

### New Workflows for Pipeline

```yaml
workflows:
  # Phase 1: Initialization
  project-init:
    stack: base
    skill: project-init.md
    status_file: PIPELINE_STATUS.md
    completion_promise: "PHASE_1_INIT_COMPLETE"
    prerequisites: []

  # Phase 2: PRD Processing
  prd-process:
    stack: base
    skill: prd-processor.md
    status_file: PRD_PROCESSING_STATUS.md
    completion_promise: "PHASE_2_PRD_COMPLETE"
    prerequisites: [PHASE_1_INIT_COMPLETE]
    outputs:
      - .claude-project/prd/*.md
      - .claude-project/docs/PROJECT_KNOWLEDGE.md

  # Phase 3: Frontend Development (Parallel with Phase 4)
  frontend-build:
    stack: react
    skill: frontend-builder.md
    status_file: SCREEN_IMPLEMENTATION_STATUS.md
    completion_promise: "PHASE_3_FRONTEND_COMPLETE"
    prerequisites: [PHASE_2_PRD_COMPLETE]
    parallel_with: [database-design]

  # Phase 3.1: Design QA (runs after frontend-build)
  design-qa:
    stack: react
    skill: design-qa.md
    status_file: SCREEN_IMPLEMENTATION_STATUS.md
    completion_promise: "PHASE_3_1_DESIGN_QA_COMPLETE"
    prerequisites: [PHASE_3_FRONTEND_COMPLETE]

  # Phase 4: Database Design (Parallel with Phase 3)
  database-design:
    stack: nestjs
    skill: database-designer.md
    status_file: DATABASE_DESIGN_STATUS.md
    completion_promise: "PHASE_4_DATABASE_COMPLETE"
    prerequisites: [PHASE_2_PRD_COMPLETE]
    parallel_with: [frontend-build]
    outputs:
      - backend/src/modules/*/entities/*.entity.ts
      - .claude-project/docs/PROJECT_DATABASE.md

  # Phase 5: Backend API Development
  backend-api:
    stack: nestjs
    skill: backend-api-builder.md
    status_file: API_IMPLEMENTATION_STATUS.md
    completion_promise: "PHASE_5_BACKEND_COMPLETE"
    prerequisites: [PHASE_4_DATABASE_COMPLETE]
    outputs:
      - backend/src/modules/*/
      - .claude-project/docs/PROJECT_API.md

  # Phase 6: Frontend API Integration
  frontend-integrate:
    stack: react
    skill: api-integrator.md
    status_file: API_INTEGRATION_STATUS.md
    completion_promise: "PHASE_6_INTEGRATE_COMPLETE"
    prerequisites:
      - PHASE_3_1_DESIGN_QA_COMPLETE
      - PHASE_5_BACKEND_COMPLETE

  # Phase 7: E2E Test Generation
  e2e-generate:
    stack: react  # Runs both frontend and backend tests
    skill: e2e-generator.md
    status_file: E2E_TEST_STATUS.md
    completion_promise: "PHASE_7_TESTS_COMPLETE"
    prerequisites: [PHASE_6_INTEGRATE_COMPLETE]

  # Phase 8: QA Loop (Iterative)
  qa-loop:
    stack: base
    skill: qa-orchestrator.md
    status_file: QA_LOOP_STATUS.md
    completion_promise: "PHASE_8_QA_COMPLETE"
    prerequisites: [PHASE_7_TESTS_COMPLETE]
    sub_workflows:
      - e2e-tests
      - design-qa
      - backend-qa
    max_iterations: 100
    pass_threshold: 95%

  # Phase 9: Deployment
  deploy:
    stack: base
    skill: deployment.md
    status_file: DEPLOYMENT_STATUS.md
    completion_promise: "PHASE_9_DEPLOY_COMPLETE"
    prerequisites: [PHASE_8_QA_COMPLETE]
    targets:
      dev: dokploy
      staging: dokploy
      production: aws

  # Phase 10: Handover Documentation
  handover:
    stack: base
    skill: handover-docs.md
    status_file: HANDOVER_STATUS.md
    completion_promise: "PHASE_10_HANDOVER_COMPLETE"
    prerequisites: [PHASE_9_DEPLOY_COMPLETE]
    outputs:
      - README.md (updated)
      - docs/DEPLOYMENT.md
      - docs/ARCHITECTURE.md
      - docs/API_REFERENCE.md
      - docs/HANDOVER.md
```

---

## Phase Execution Flow (Visual)

```
                            PIPELINE START
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │  Phase 1: project-init  │
                    │  /ralph project-init    │
                    └───────────┬─────────────┘
                                │
                                ▼
                    ┌─────────────────────────┐
                    │  Phase 2: prd-process   │
                    │  /ralph prd-process     │
                    └───────────┬─────────────┘
                                │
              ┌─────────────────┴─────────────────┐
              │           PARALLEL GATE           │
              ▼                                   ▼
┌─────────────────────────┐         ┌─────────────────────────┐
│  Phase 3: frontend-build│         │ Phase 4: database-design│
│  /ralph frontend-build  │         │ /ralph database-design  │
└───────────┬─────────────┘         └───────────┬─────────────┘
            │                                   │
            ▼                                   │
┌─────────────────────────┐                     │
│ Phase 3.1: design-qa    │                     │
│ /ralph design-qa        │                     │
└───────────┬─────────────┘                     │
            │                                   │
            │         ┌─────────────────────────┘
            │         │
            ▼         ▼
┌─────────────────────────┐
│  PARALLEL SYNC GATE     │
│  Wait for both:         │
│  - PHASE_3_1_COMPLETE   │
│  - PHASE_4_COMPLETE     │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Phase 5: backend-api   │
│  /ralph backend-api     │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ Phase 6: frontend-int   │
│ /ralph frontend-integrate│
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Phase 7: e2e-generate  │
│  /ralph e2e-generate    │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────────┐
│           Phase 8: qa-loop                   │
│  ┌─────────────────────────────────────┐    │
│  │         QA Iteration Loop           │    │
│  │                                     │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────┐ │    │
│  │  │ e2e-test│─►│design-qa│─►│ fix │ │    │
│  │  └─────────┘  └─────────┘  └──┬──┘ │    │
│  │       ▲                       │    │    │
│  │       └───────────────────────┘    │    │
│  │                                     │    │
│  │  Exit when: pass_rate >= 95%       │    │
│  └─────────────────────────────────────┘    │
└───────────┬─────────────────────────────────┘
            │
            ▼
┌─────────────────────────┐
│  Phase 9: deploy        │
│  /ralph deploy          │
│  ├── Dev → Dokploy      │
│  ├── Staging → Dokploy  │
│  └── Prod → AWS         │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│  Phase 10: handover     │
│  /ralph handover        │
└───────────┬─────────────┘
            │
            ▼
                    PIPELINE COMPLETE
```

---

## Master Orchestrator Design

### `/fullstack` Command

```markdown
---
description: Full-stack development pipeline orchestrator (Ralph-powered)
argument-hint: "[command] [options]"
---

# Full-Stack Pipeline Orchestrator

## Commands

/fullstack                    # Show status, auto-run next phase(s)
/fullstack status            # Detailed pipeline status
/fullstack run               # Start/continue pipeline (auto mode)
/fullstack run --parallel    # Run parallel phases concurrently
/fullstack phase <name>      # Run specific phase
/fullstack reset <phase>     # Reset phase to pending

## Phase Names

init, prd, frontend, design-qa, database, backend, integrate, test, qa, deploy, handover

## Execution Logic

### Step 1: Read Pipeline Status
Read `.claude-project/PIPELINE_STATUS.md` to determine:
- Current phase(s) in progress
- Completed phases
- Blocked phases (prerequisites not met)

### Step 2: Determine Runnable Phases
A phase is runnable if:
- All prerequisites are COMPLETE
- Phase status is PENDING or FAILED
- No blocking errors exist

### Step 3: Execute Phases
For each runnable phase:
1. Set status to IN_PROGRESS
2. Launch Ralph workflow: `/ralph <workflow> <project>`
3. Ralph executes skill with its own status file
4. On completion, update PIPELINE_STATUS.md
5. Check for newly runnable phases

### Step 4: Parallel Execution
For phases marked `parallel_with`:
1. Launch both workflows simultaneously
2. Track both in PIPELINE_STATUS.md
3. Wait for sync gate before continuing

### Step 5: QA Loop (Phase 8)
Special handling:
1. Run e2e-tests workflow
2. Check pass rate
3. If < 95%, run fix-issues skill
4. Run design-qa workflow
5. Repeat until pass_rate >= 95% OR max_iterations reached
6. If blocked, output report and mark for manual review

### Step 6: Deployment (Phase 9)
Sequential deployment:
1. Deploy to dev (Dokploy) → Verify health
2. Deploy to staging (Dokploy) → Verify health
3. Deploy to production (AWS) → Verify health
```

---

## Status File Design

### `.claude-project/PIPELINE_STATUS.md`

```markdown
# Development Pipeline Status

**Project:** $PROJECT_NAME
**Started:** 2025-01-08
**Last Updated:** 2025-01-10 14:32:00

## Pipeline Overview

| Phase | Workflow | Status | Progress | Duration |
|-------|----------|--------|----------|----------|
| 1 | project-init | ✅ COMPLETE | 100% | 2m |
| 2 | prd-process | ✅ COMPLETE | 100% | 8m |
| 3 | frontend-build | 🔄 IN_PROGRESS | 67% | 45m |
| 3.1 | design-qa | ⏳ BLOCKED | 0% | - |
| 4 | database-design | ✅ COMPLETE | 100% | 12m |
| 5 | backend-api | 🔄 IN_PROGRESS | 34% | 28m |
| 6 | frontend-integrate | ⏳ BLOCKED | 0% | - |
| 7 | e2e-generate | ⏳ PENDING | 0% | - |
| 8 | qa-loop | ⏳ PENDING | 0% | - |
| 9 | deploy | ⏳ PENDING | 0% | - |
| 10 | handover | ⏳ PENDING | 0% | - |

## Status Legend

- ✅ COMPLETE - Phase finished successfully
- 🔄 IN_PROGRESS - Phase currently running
- ⏳ PENDING - Ready to run (prerequisites met)
- ⛔ BLOCKED - Waiting for prerequisites
- ❌ FAILED - Phase failed, needs attention
- ⏸️ PAUSED - Manually paused

## Parallel Execution Status

```
Phase 3 (frontend-build) ████████████████░░░░░░░░ 67%
Phase 4 (database-design) ████████████████████████ 100% ✓
                          ─────────────────────────────
                          Waiting for Phase 3 to sync
```

## Phase Details

### Phase 1: project-init ✅
- **Completed:** 2025-01-08 09:00
- **Outputs:**
  - [x] .claude-project/ folder created
  - [x] Git repository initialized
  - [x] .claude submodule configured

### Phase 2: prd-process ✅
- **Completed:** 2025-01-08 09:15
- **Outputs:**
  - [x] .claude-project/prd/prd.md
  - [x] .claude-project/docs/PROJECT_KNOWLEDGE.md
  - [x] .claude-project/plans/ structure

### Phase 3: frontend-build 🔄
- **Started:** 2025-01-09 10:00
- **Progress:** 32/48 screens
- **Status File:** SCREEN_IMPLEMENTATION_STATUS.md
- **Current Item:** Dashboard > PatientList

### Phase 3.1: design-qa ⛔
- **Blocked By:** Phase 3 (frontend-build)
- **Will Run After:** PHASE_3_FRONTEND_COMPLETE

### Phase 4: database-design ✅
- **Completed:** 2025-01-09 11:30
- **Progress:** 12/12 entities
- **Outputs:**
  - [x] All entity files created
  - [x] Migrations generated
  - [x] PROJECT_DATABASE.md updated

### Phase 5: backend-api 🔄
- **Started:** 2025-01-09 11:45
- **Progress:** 17/50 endpoints
- **Status File:** API_IMPLEMENTATION_STATUS.md
- **Current Module:** users

## Completion Promises

| Promise | Status | Timestamp |
|---------|--------|-----------|
| PHASE_1_INIT_COMPLETE | ✅ | 2025-01-08 09:00 |
| PHASE_2_PRD_COMPLETE | ✅ | 2025-01-08 09:15 |
| PHASE_3_FRONTEND_COMPLETE | ⏳ | - |
| PHASE_3_1_DESIGN_QA_COMPLETE | ⏳ | - |
| PHASE_4_DATABASE_COMPLETE | ✅ | 2025-01-09 11:30 |
| PHASE_5_BACKEND_COMPLETE | ⏳ | - |
| PHASE_6_INTEGRATE_COMPLETE | ⏳ | - |
| PHASE_7_TESTS_COMPLETE | ⏳ | - |
| PHASE_8_QA_COMPLETE | ⏳ | - |
| PHASE_9_DEPLOY_COMPLETE | ⏳ | - |
| PHASE_10_HANDOVER_COMPLETE | ⏳ | - |

## Error Log

No errors currently.

## Execution History

| Timestamp | Event | Details |
|-----------|-------|---------|
| 2025-01-10 14:32 | Phase 5 progress | 17/50 endpoints (34%) |
| 2025-01-10 14:15 | Phase 3 progress | 32/48 screens (67%) |
| 2025-01-09 11:45 | Phase 5 started | backend-api workflow |
| 2025-01-09 11:30 | Phase 4 complete | 12 entities created |
| 2025-01-09 10:00 | Parallel start | Phases 3 & 4 |
```

---

## Deployment Strategy

### Phase 9: Deployment Skill

```yaml
deployment:
  environments:
    dev:
      platform: dokploy
      url: dev.myapp.com
      auto_deploy: true
      health_check: /api/health

    staging:
      platform: dokploy
      url: staging.myapp.com
      auto_deploy: true
      health_check: /api/health
      requires: dev_healthy

    production:
      platform: aws
      service: ecs  # or lambda, ec2
      url: myapp.com
      auto_deploy: false  # Requires explicit approval in real use
      health_check: /api/health
      requires: staging_healthy

  docker:
    backend:
      dockerfile: backend/Dockerfile
      registry: ecr  # or dockerhub

    frontend:
      dockerfile: frontend/Dockerfile
      registry: ecr

  verification:
    - health_check
    - smoke_tests
    - rollback_on_failure
```

### Dokploy Integration

```bash
# Deploy to Dokploy (dev/staging)
dokploy deploy --project $PROJECT_NAME --env dev
dokploy deploy --project $PROJECT_NAME --env staging
```

### AWS Production Deployment

```bash
# Build and push to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
docker build -t $PROJECT_NAME-backend ./backend
docker tag $PROJECT_NAME-backend:latest $ECR_REGISTRY/$PROJECT_NAME-backend:latest
docker push $ECR_REGISTRY/$PROJECT_NAME-backend:latest

# Deploy via ECS/EKS or App Runner
aws ecs update-service --cluster $CLUSTER --service $SERVICE --force-new-deployment
```

---

## Implementation Files to Create

### 1. Master Command
**File:** `.claude/base/commands/dev/fullstack.md`

### 2. Pipeline Status Template
**File:** `.claude/base/templates/claude-project/PIPELINE_STATUS.template.md`

### 3. New Ralph Workflows (Skills)

| Skill | Path | Purpose |
|-------|------|---------|
| project-init | `.claude/base/skills/project-init.md` | Phase 1 orchestration |
| prd-processor | `.claude/base/skills/prd-processor.md` | Phase 2 orchestration |
| frontend-builder | `.claude/react/skills/frontend-builder.md` | Phase 3 orchestration |
| database-designer | `.claude/nestjs/skills/database-designer.md` | Phase 4 orchestration |
| backend-api-builder | `.claude/nestjs/skills/backend-api-builder.md` | Phase 5 orchestration |
| api-integrator | `.claude/react/skills/api-integrator.md` | Phase 6 orchestration |
| e2e-generator | `.claude/base/skills/e2e-generator.md` | Phase 7 orchestration |
| qa-orchestrator | `.claude/base/skills/qa-orchestrator.md` | Phase 8 (meta-loop) |
| deployment | `.claude/base/skills/deployment.md` | Phase 9 |
| handover-docs | `.claude/base/skills/handover-docs.md` | Phase 10 |

### 4. Status File Templates

| Template | Path |
|----------|------|
| PIPELINE_STATUS | `.claude/base/templates/claude-project/PIPELINE_STATUS.template.md` |
| PRD_PROCESSING_STATUS | `.claude/base/templates/ralph/prd-processing-status.template.md` |
| DATABASE_DESIGN_STATUS | `.claude/base/templates/ralph/database-design-status.template.md` |
| QA_LOOP_STATUS | `.claude/base/templates/ralph/qa-loop-status.template.md` |
| DEPLOYMENT_STATUS | `.claude/base/templates/ralph/deployment-status.template.md` |
| HANDOVER_STATUS | `.claude/base/templates/ralph/handover-status.template.md` |

### 5. Update Existing Files

| File | Changes |
|------|---------|
| `.claude/base/commands/dev/ralph.md` | Add new workflows to registry |
| `.claude/base/templates/claude-project/plans/` | Add phase tracking columns |

---

## Usage Examples

### Start Fresh Project
```bash
/fullstack init my-new-project
# Creates project, runs Phase 1, outputs PIPELINE_STATUS.md
```

### Continue Pipeline
```bash
/fullstack run
# Reads PIPELINE_STATUS.md
# Determines runnable phases
# Launches Ralph workflows
# Updates status on completion
```

### Run Specific Phase
```bash
/fullstack phase backend
# Runs Phase 5 only (checks prerequisites first)
```

### View Status
```bash
/fullstack status
# Displays formatted PIPELINE_STATUS.md with progress bars
```

### Reset Failed Phase
```bash
/fullstack reset design-qa
# Resets Phase 3.1 to PENDING, clears errors
```

### Run with Parallel Execution
```bash
/fullstack run --parallel
# Runs Phases 3 & 4 simultaneously when ready
```

---

## QA Loop Design (Phase 8)

The QA loop is a **meta-workflow** that orchestrates multiple Ralph workflows:

```
┌────────────────────────────────────────────────────────────┐
│                    QA Loop Orchestrator                     │
│                                                            │
│  Iteration 1:                                              │
│  ├── /ralph e2e-tests frontend → 45/50 pass (90%)        │
│  ├── Analyze failures → Generate fix plan                 │
│  ├── Apply fixes to 5 failing tests                       │
│  └── /ralph design-qa frontend → 44/48 pass (92%)        │
│                                                            │
│  Iteration 2:                                              │
│  ├── /ralph e2e-tests frontend → 48/50 pass (96%)        │
│  ├── /ralph design-qa frontend → 47/48 pass (98%)        │
│  └── Combined pass rate: 97% ✓                           │
│                                                            │
│  Exit: pass_rate >= 95%                                   │
│  Output: QA_LOOP_STATUS.md with all results               │
└────────────────────────────────────────────────────────────┘
```

---

## Summary

### What We're Building

1. **`/fullstack` command** - Master orchestrator that manages the entire pipeline
2. **10 Ralph workflows** - One for each phase, all autonomous
3. **Parallel execution** - Phases 3 & 4 run concurrently
4. **Zero human checkpoints** - Status file gates control flow
5. **QA loop** - Iterates until 95% pass rate
6. **Multi-env deployment** - Dokploy (dev/staging) → AWS (production)
7. **Complete handover docs** - Generated automatically at the end

### Key Benefits

- **Resumable**: Stop anytime, resume from checkpoint
- **Observable**: PIPELINE_STATUS.md shows all progress
- **Autonomous**: Ralph handles each phase independently
- **Scalable**: Add new phases by adding workflows
- **Debuggable**: Each phase has its own status file

---

## Next Steps

1. ✅ Plan approved
2. Create `/fullstack` command
3. Create PIPELINE_STATUS.template.md
4. Create phase-specific skills
5. Update ralph.md with new workflows
6. Create deployment skill (Dokploy + AWS)
7. Test with sample project
