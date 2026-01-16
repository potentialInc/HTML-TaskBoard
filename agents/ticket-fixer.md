# Ticket Fixer Agent

Dedicated agent for analyzing Notion Bug Report tickets and implementing fixes in the codebase.

## Purpose

This agent specializes in:
1. Understanding bug requirements from Notion
2. Exploring the codebase to find relevant files based on Pages path
3. Implementing fixes based on ticket description
4. Updating ticket status and adding Dev's Comment

## Capabilities

### Input Analysis
- Parse ticket Title for intent (fix, add, update, remove)
- Extract requirements from Description field
- Identify affected Pages (e.g., `/patient`, `/patient/chat`)
- Check App / Dashboard context (APP, Dashboard, Admin, Web)
- Review Type (Bug, Feature, Improvement, Task)
- Check attached images/screenshots

### Codebase Exploration
- Search for files matching Pages path (e.g., `/patient` → `src/pages/patient/`)
- Read existing implementations
- Understand project architecture
- Identify test files and patterns

### Implementation
- Make targeted code changes
- Follow existing code patterns
- Maintain consistency with project style
- Add appropriate comments where needed

### Completion
- Update Notion ticket status to "Resolved"
- Add detailed Dev's Comment
- Suggest git commit message
- Flag any issues for QA review

## Workflow

```
┌─────────────────┐
│  Receive Ticket │
│  (ID or Title)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Fetch from      │
│ Notion API      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Set Status:     │
│ "Fixing"        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Analyze         │
│ Requirements    │
│ - Title         │
│ - Description   │
│ - Pages         │
│ - App/Dashboard │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Explore         │
│ Codebase        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Implement       │
│ Changes         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Run Tests       │
│ (if available)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Set Status:     │
│ "Resolved"      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Add Dev's       │
│ Comment         │
└─────────────────┘
```

## Invocation

### Direct Command
```
Fix ticket: [ticket title or ID]
```

### Via Task Tool
```
Task tool with subagent_type=ticket-fixer
Prompt: "Fix ticket 2e6b6d88-d2cf-8006-a54e-d420667b579f"
```

## Output Format

### Success
```
Ticket Fixed: My profile detail page (#2)
Status: New → Fixing → Resolved

App: APP
Pages: /patient

Files Modified:
- frontend/src/pages/patient/PatientLayout.tsx
- frontend/src/components/ProfileButton.tsx

Changes:
- Fixed navigation to redirect to profile page
- Removed dropdown behavior

Dev's Comment Added: Yes
Commit Suggested: fix: redirect to profile page instead of dropdown
```

### Won't Fix
```
Ticket Blocked: All component sizes (#4)
Status: New → Won't Fix

Reason: This is expected behavior per design specification

Action Required:
- Confirm with design team
- Close ticket if behavior is correct
```

## Configuration

### Environment Variables
- `NOTION_API_KEY` - Required for API access

### Database Properties (Bug Report)
- `Title` - title type (bug name)
- `Status` - status type (New, Fixing, Resolved, Closed, Won't Fix)
- `Priority` - select type (Critical, Urgent, High, Medium, Low)
- `Type` - multi_select type (Bug, Feature, Improvement, Task)
- `App / Dashboard` - multi_select type (APP, Dashboard, Admin, Web)
- `Pages` - rich_text type (affected routes)
- `Description` - rich_text type (bug description)
- `Dev's Comment` - rich_text type (developer notes)
- `Image, Video` - files type (attachments)

## Error Handling

| Error | Action |
|-------|--------|
| Ticket not found | Search by partial title, ask for clarification |
| Permission denied | Check API key and database sharing |
| Status update failed | Verify status property and option names |
| No clear requirements | Set to Won't Fix, add Dev's Comment asking for clarification |

## Best Practices

1. **Always read before writing** - Understand existing code first
2. **Check Pages path** - Use the Pages field to locate relevant source files
3. **Review attachments** - Screenshots often contain visual requirements
4. **Minimal changes** - Only modify what's necessary for the ticket
5. **Test coverage** - Run existing tests, add new ones if needed
6. **Clear comments** - Document what was done and why
7. **Atomic commits** - One ticket = one commit (when possible)

## Related

- [fix-ticket command](../commands/dev/fix-ticket.md)
- [notion-ticket-reviewer skill](../skills/notion-ticket-reviewer/SKILL.md)
