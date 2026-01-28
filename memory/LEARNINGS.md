# Team Learnings

Shared learnings across all projects using this .claude base configuration.
These are team-wide conventions and patterns.

---

## Scope

This file contains learnings that apply to **ALL projects** using this base configuration:
- Team coding conventions
- Shared library preferences
- Common architectural patterns
- Team process preferences

For project-specific learnings, use `.claude-project/memory/LEARNINGS.md`
For personal preferences, use `~/.claude/memory/LEARNINGS.md`

---

## Entries

<!-- Team learnings are appended below this line -->

### 2026-01-28: Google OAuth Integration

**Type**: guideline-reference
**Source**: Session reflection - design-flow project

**Description**: Full Google OAuth integration guide created.

**See**: [guidelines/google-oauth.md](../guidelines/google-oauth.md)

Key gotchas documented:
- Use localStorage for OAuth state (not sessionStorage)
- Backend SocialLoginTypeEnum uses numeric values (GOOGLE=3)
- Access tokens validate via userinfo endpoint
- New users require `termsAndConditionsAccepted: true`

---
