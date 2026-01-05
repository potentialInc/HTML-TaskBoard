---
description: Update dev documentation before context compaction
argument-hint: Optional - specific docs to focus on (e.g., "API", "database")
---

## Purpose

Update `.claude/docs/` before context reset to preserve session learnings.

**Priority**: Focus on capturing information that would be hard to rediscover or reconstruct from code alone.

## Documentation Files to Update

Review and update the following files in `.claude/docs/` based on work done this session:

### 1. PROJECT_KNOWLEDGE.md

Update with:
- Architecture decisions made
- New patterns discovered
- Integration points found
- Business rules clarified
- Key data entities added/modified
- Common workflows documented

### 2. PROJECT_API.md

Update with:
- New endpoints added (with full documentation)
- Endpoint modifications
- Request/response format changes
- Authentication requirement updates
- New modules or controllers

### 3. PROJECT_API_INTEGRATION.md

Update with:
- Screen-to-API mapping changes
- Implementation status updates (use: ✅ DONE, ⚠️ PARTIAL, ❌ TODO)
- New frontend pages and their API dependencies
- Integration issues discovered

### 4. PROJECT_DATABASE.md

Update with:
- New entities or columns
- Relationship changes
- Index additions/modifications
- Migration notes
- Design decisions with reasoning

### 5. BEST_PRACTICES.md

Update with:
- New patterns discovered (with ✅ GOOD / ❌ BAD examples)
- Anti-patterns found during debugging
- Code examples that solved tricky problems
- Performance or security insights

### 6. TROUBLESHOOTING.md

Update with:
- Issues encountered and their solutions
- Debug techniques that worked
- Error messages and fixes
- Common pitfalls to avoid

## Session Context to Capture

Include relevant information about:
- Complex problems solved
- Tricky bugs found and fixed
- Performance optimizations made
- Security considerations addressed
- Testing approaches used

## Handoff Notes

For seamless continuation after context reset:
- Current work in progress (exact file and line if applicable)
- Uncommitted changes that need attention
- Next immediate steps
- Commands to run on restart
- Any temporary workarounds needing permanent fixes

## Additional Focus: $ARGUMENTS
