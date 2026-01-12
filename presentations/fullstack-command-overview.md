---
title: Fullstack Pipeline Orchestrator
subtitle: Team Training Guide
author: Potential Inc
date: 2026-01-12
---

# Slide 1: Welcome
- Fullstack Pipeline Orchestrator
- How to go from project setup to deployment
- One command to rule them all

# Slide 2: What is /fullstack?
- A skill-chain orchestrator
- Runs the full development lifecycle
- Tracks progress in PIPELINE_STATUS.md
- Delegates to specialized skills per phase

# Slide 3: The 9 Pipeline Phases
- init: Project setup (.claude-project/, .claude/)
- prd: Convert PRD to PROJECT_KNOWLEDGE.md
- database: Design entities and migrations
- backend: Implement API endpoints
- frontend: Build React screens
- integrate: Connect frontend to backend
- test: Generate E2E tests
- qa: Run design QA (95% pass required)
- ship: Deploy to production

# Slide 4: Phase Flow
- Each phase has prerequisites
- init has no prerequisites
- prd requires init
- database requires prd
- backend requires database
- frontend requires prd (parallel with backend)
- integrate requires backend + frontend
- test requires integrate
- qa requires test
- ship requires qa

# Slide 5: Quick Start Commands
- Show status: /fullstack my-project
- Run next phase: /fullstack my-project --run
- Run specific: /fullstack my-project --phase backend
- Run all: /fullstack my-project --run-all
- Reset phase: /fullstack my-project --reset database

# Slide 6: Tech Stack Configuration
- Configured in PIPELINE_STATUS.md
- Backend: nestjs or django
- Frontends: react, react-native
- Resolves skill paths automatically
- Example: nestjs + react-native

# Slide 7: Frontend Multi-Path
- Three implementation paths available
- Design from scratch: PRD to design prompts
- Convert from Figma: Use MCP tools
- Convert from HTML: Existing templates
- Agent asks which path to use

# Slide 8: Status Tracking
- PIPELINE_STATUS.md tracks everything
- Status icons: Pending, In Progress, Complete, Failed
- Output column shows deliverables
- Execution log records all runs
- Never lose progress

# Slide 9: Ralph Integration
- Some phases delegate to Ralph
- Item-level iteration (48 screens, 29 tests)
- Ralph runs until completion criteria met
- QA phase needs 95% pass rate
- Combines /ralph e2e-tests and design-qa

# Slide 10: Example Workflow
- /fullstack my-app --run (init)
- /fullstack my-app --run (prd)
- /fullstack my-app --run (database)
- Continue until ship
- Or use --run-all for automation

# Slide 11: Key Takeaways
- One command orchestrates everything
- 9 phases from init to ship
- Tracks progress automatically
- Supports multiple tech stacks
- Delegates to Ralph for iteration
- Just run /fullstack and follow along
