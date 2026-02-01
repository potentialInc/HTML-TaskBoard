---
description: Session analysis with multi-agent pipeline - docs, automation, learnings, follow-ups
argument-hint: [optional: "quick" for single-pass, or focus area like "backend", "frontend"]
allowed-tools: Bash(git *), Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
---

# Reflect Command (Enhanced)

Comprehensive session wrap-up with multi-agent analysis. Proposes documentation updates, automation opportunities, learnings for memory, and follow-up tasks.

---

## Mode Selection

Check `$ARGUMENTS`:

- **"quick"** → Run Quick Mode (original single-pass analysis)
- **Empty or other** → Run Full Mode (multi-agent pipeline)

---

## Quick Mode (Single-Pass)

For short sessions or when you want fast learning capture only.

### Step 1: Analyze Conversation

Scan for correction signals, preferences, and patterns:

| Signal Type | Examples | Confidence |
|-------------|----------|------------|
| Explicit correction | "No, don't do X" | HIGH |
| Direct preference | "I prefer X over Y" | HIGH |
| Partial correction | "That's mostly right, but..." | MEDIUM |
| One-time adjustment | Single instance change | LOW |

### Step 2: Present Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 REFLECTION ANALYSIS (Quick Mode)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

HIGH CONFIDENCE:
1. [Learning title]
   Signal: "[user message]"
   Proposed: Add to [LEARNINGS/PREFERENCES/CORRECTIONS].md

MEDIUM CONFIDENCE:
1. [Learning title]
   Note: Consider if pattern repeats

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 3: Ask Memory Level and Save

Use AskUserQuestion for memory level selection, then save to appropriate tier.

---

## Full Mode (Multi-Agent Pipeline)

For substantial sessions. Runs 5 specialized agents in 2 phases.

```
┌─────────────────────────────────────────────────────┐
│  1. Git Status Check                                │
├─────────────────────────────────────────────────────┤
│  2. Phase 1: 4 Analysis Agents (PARALLEL)           │
│     ┌─────────────────┬─────────────────┐           │
│     │  doc-updater    │  automation-    │           │
│     │  (docs update)  │  scout          │           │
│     ├─────────────────┼─────────────────┤           │
│     │  learning-      │  followup-      │           │
│     │  extractor      │  suggester      │           │
│     └─────────────────┴─────────────────┘           │
├─────────────────────────────────────────────────────┤
│  3. Phase 2: Validation (SEQUENTIAL)                │
│     ┌───────────────────────────────────┐           │
│     │       duplicate-checker           │           │
│     └───────────────────────────────────┘           │
├─────────────────────────────────────────────────────┤
│  4. Integrate Results & Present Options             │
├─────────────────────────────────────────────────────┤
│  5. Execute Selected Actions                        │
└─────────────────────────────────────────────────────┘
```

### Step 1: Git Status Check

```bash
git status --short
git diff --stat HEAD~3 2>/dev/null || git diff --stat
```

### Step 2: Create Session Summary

Before launching agents, create a summary to share:

```markdown
## Session Summary

**Work Performed:**
- [Main tasks completed]
- [Files created/modified]
- [Key decisions made]

**Session Context:**
- Duration: [approximate]
- Focus area: [if specified in $ARGUMENTS]
```

### Step 3: Phase 1 - Launch 4 Agents in PARALLEL

**CRITICAL: Launch all 4 agents in a SINGLE message with 4 Task tool calls.**

```
Task(
    subagent_type="general-purpose",
    description="Documentation analysis",
    prompt="You are the doc-updater agent. [Session Summary]\n\nAnalyze this session and propose specific updates to CLAUDE.md and context.md. Follow the instructions in .claude/base/agents/doc-updater.md"
)

Task(
    subagent_type="general-purpose",
    description="Automation detection",
    prompt="You are the automation-scout agent. [Session Summary]\n\nAnalyze this session for repetitive patterns worth automating. Follow the instructions in .claude/base/agents/automation-scout.md"
)

Task(
    subagent_type="general-purpose",
    description="Learning extraction",
    prompt="You are the learning-extractor agent. [Session Summary]\n\nExtract learnings, mistakes, and discoveries from this session. Follow the instructions in .claude/base/agents/learning-extractor.md"
)

Task(
    subagent_type="general-purpose",
    description="Follow-up suggestions",
    prompt="You are the followup-suggester agent. [Session Summary]\n\nIdentify incomplete work and prioritize follow-up tasks. Follow the instructions in .claude/base/agents/followup-suggester.md"
)
```

### Step 4: Phase 2 - Validation (Sequential)

After Phase 1 completes, run duplicate-checker with the results:

```
Task(
    subagent_type="general-purpose",
    model="haiku",
    description="Duplicate validation",
    prompt="You are the duplicate-checker agent. Validate these Phase 1 proposals for duplicates.\n\n## doc-updater proposals:\n[results]\n\n## automation-scout proposals:\n[results]\n\nFollow the instructions in .claude/base/agents/duplicate-checker.md"
)
```

### Step 5: Integrate and Present Results

Combine all agent outputs into a unified summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 REFLECTION ANALYSIS (Full Mode)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Documentation Updates
[doc-updater summary]
Validation: [duplicate-checker feedback]

## Automation Opportunities
[automation-scout summary]
Validation: [duplicate-checker feedback]

## Learnings Captured
[learning-extractor summary]
- HIGH confidence: [X]
- MEDIUM confidence: [X]

## Follow-up Tasks
[followup-suggester summary]
- P0 (Urgent): [X]
- P1 (High): [X]
- P2 (Medium): [X]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Step 6: Action Selection

Use AskUserQuestion with multiSelect:

```
AskUserQuestion(
    questions=[{
        "question": "Which actions would you like to perform?",
        "header": "Actions",
        "multiSelect": true,
        "options": [
            {"label": "Save learnings to memory", "description": "Add HIGH confidence learnings to memory files"},
            {"label": "Update documentation", "description": "Apply approved doc updates to CLAUDE.md/context.md"},
            {"label": "Create automation", "description": "Generate skill/command/agent files"},
            {"label": "Create follow-up tasks", "description": "Save task list for next session"}
        ]
    }, {
        "question": "Which memory level for learnings?",
        "header": "Memory Level",
        "multiSelect": false,
        "options": [
            {"label": "Project (Recommended)", "description": ".claude-project/memory/ - this project only"},
            {"label": "Team", "description": ".claude/base/memory/ - shared via submodule"},
            {"label": "Personal", "description": "~/.claude/memory/ - follows you everywhere"}
        ]
    }]
)
```

### Step 7: Execute Selected Actions

Based on user selection:

#### Save Learnings
Append to appropriate memory file based on level:
- `LEARNINGS.md` - General learnings
- `PREFERENCES.md` - Explicit preferences
- `CORRECTIONS.md` - Mistakes to avoid

Format each entry:
```markdown
### [Date]: [Learning Title]

**Type**: [correction|preference|pattern|discovery]
**Confidence**: [HIGH|MEDIUM]
**Source**: Session reflection

**Description**: [What was learned]

**Evidence**:
- "[Quoted signal]"

---
```

#### Update Documentation
Apply approved updates from doc-updater (respecting duplicate-checker validation).

#### Create Automation
Generate files for approved automations from automation-scout.

#### Create Follow-up Tasks
Save task list to `.claude-project/status/followup-tasks.md`.

### Step 8: Commit Changes

If changes were made:

```bash
git add .claude-project/memory/ .claude-project/status/
git commit -m "reflect: Session wrap-up

Captured:
- [X] learnings
- [X] documentation updates
- [X] follow-up tasks

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
```

---

## Focus Area Filtering

If `$ARGUMENTS` contains a focus area (not "quick"), pass it to agents:

| Focus | Filter to |
|-------|-----------|
| `backend` | NestJS, API, database, services |
| `frontend` | React, UI, components, styling |
| `testing` | Tests, fixtures, assertions |
| `git` | Commit, PR, branch conventions |

---

## Memory File Locations (3-Tier System)

| Level | Location | Use Case |
|-------|----------|----------|
| Personal | `~/.claude/memory/` | Individual preferences |
| Team | `.claude/base/memory/` | Shared conventions |
| Project | `.claude-project/memory/` | Project-specific |

Each contains: `LEARNINGS.md`, `PREFERENCES.md`, `CORRECTIONS.md`

---

## Error Handling

- If an agent fails, continue with others and note the failure
- If duplicate-checker fails, default to "Approved" for all proposals
- If no learnings found, report "No significant learnings detected"

---

## Example Usage

```bash
# Full multi-agent analysis (default)
/reflect

# Quick single-pass mode
/reflect quick

# Focus on backend patterns
/reflect backend

# Focus on frontend patterns
/reflect frontend
```
