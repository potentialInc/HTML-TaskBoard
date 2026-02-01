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

**See**: [guides/google-oauth.md](../guides/google-oauth.md)

Key gotchas documented:
- Use localStorage for OAuth state (not sessionStorage)
- Backend SocialLoginTypeEnum uses numeric values (GOOGLE=3)
- Access tokens validate via userinfo endpoint
- New users require `termsAndConditionsAccepted: true`

---

### 2026-01-28: Use Dynamic Service Discovery for PM2 Configs

**Type**: correction
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: Don't hardcode dashboard services in PM2/Docker configs. Use dynamic discovery pattern instead because different projects have different dashboard combinations (admin, coach, owner, etc.).

**Evidence**:
- "there must be multiple dashboard so it shouldn't be hardcoded"
- "some time there is frontend-admin-dashboard frontend-coach-dashboard frontend-owner-dashboard"

**Pattern**: Auto-discover `frontend-*-dashboard/` folders at runtime rather than listing each explicitly. This makes configs portable across projects.

---

### 2026-01-28: Vite/React Router Requires Explicit --port Flag

**Type**: technical-learning
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: Vite dev server (used by React Router v7) does NOT respect the `PORT` environment variable. Must pass `--port` flag explicitly via CLI.

**Correct approach**:
```javascript
// In PM2 ecosystem.config.js
args: `run dev -- --port ${port}`,  // ✓ Works
```

**Incorrect approach**:
```javascript
env: {
  PORT: 5173,  // ✗ Ignored by Vite
}
```

---

### 2026-01-28: Always Create Symlinks When Adding Commands to base/commands

**Type**: correction
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: When creating new commands in `.claude/base/commands/`, you must also create a corresponding symlink in `.claude/commands/` for the command to be accessible.

**Evidence**:
- "I think you forgot symlink. I'm not sure how you create a command, whenever you create a command can you check symlink?"

**Pattern**:
```bash
# After creating .claude/base/commands/dev/start.md
cd .claude/commands/dev
ln -s ../../base/commands/dev/start.md start.md
```

**Structure**:
```
.claude/
├── base/commands/dev/start.md     # Actual command file
└── commands/dev/start.md          # Symlink → ../../base/commands/dev/start.md
```

**Checklist for new commands**:
1. Create command file in `.claude/base/commands/<category>/<name>.md`
2. Create symlink in `.claude/commands/<category>/<name>.md`
3. Verify with `ls -la .claude/commands/<category>/<name>.md`

---

### 2026-02-01: System Prompt Token Cost Dominates .claude Directory Impact

**Type**: discovery
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: The primary performance cost of a large `.claude` directory is NOT disk space or startup time, but the skills list rendered in the system prompt. This consumes tokens on every turn of conversation.

**Key insight**: With 87 skills, each conversation starts with ~2-3K tokens just for the skills list. Reducing unused skills has immediate context savings.

**Optimization target**: Minimize skills enumerated in system prompt, not file count.

---

### 2026-02-01: Hybrid Architecture for .claude - Domain-Specific Submodules

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project (claude-marketing extraction)

**Description**: Splitting a monolithic skills repository into core base + functional domain repos (as git submodules) effectively reduces context bloat while maintaining cohesion.

**Architecture**:
```
.claude/
├── base/           ← Core skills only (Tier 1)
├── react/          ← Framework-specific (Tier 2)
├── nestjs/         ← Framework-specific (Tier 2)
├── marketing/      ← Domain-specific (Tier 2, optional)
└── operations/     ← Domain-specific (Tier 2, future)
```

**Benefits**:
- Dev projects don't load marketing skills
- Marketing projects can add the submodule when needed
- Each domain evolves independently

**Evidence**: Extracted 23 marketing skills (12,694 lines) to separate repo, reducing base complexity.

---

### 2026-02-01: Follow Existing Patterns When Refactoring Architecture

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: When refactoring architecture, adhering to existing patterns in the codebase makes transitions smoother. The marketing extraction followed the same submodule pattern used for react/nestjs/django.

**Evidence**: "Following existing patterns (tech stack submodules) made the refactor smooth"

**Application**: Before creating new architecture, check if similar patterns exist and follow them.

---

### 2026-02-01: HTTPS Git Clone as SSH Fallback

**Type**: technical-learning
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: When SSH authentication fails due to permissions issues, git clone via HTTPS provides a reliable alternative.

```bash
# If this fails with "Permission denied (publickey)"
git clone git@github.com:org/repo.git

# Use HTTPS instead
git clone https://github.com/org/repo.git
```

**Note**: HTTPS may prompt for credentials or use credential helper.

---

### 2026-02-01: Measurable Impact of Domain Skill Extraction

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: Extracting domain-specific skills to separate repos has quantifiable benefits:

| Metric | Before | After |
|--------|--------|-------|
| Marketing skills in base | 23 | 0 |
| Lines removed from base | - | 12,694 |
| Base skill count | 30 | 7 |

**Implication**: Similar extractions (operations, dev tools) can yield comparable reductions.

---

### 2026-02-01: Submodule Integration Preserves Accessibility

**Type**: discovery
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: Adding extracted domain repos as git submodules (rather than separate npm packages or dependencies) maintains seamless access within the project context.

**Advantages**:
- Skills appear in same `.claude/` folder structure
- No version management complexity
- Easy to add/remove based on project needs
- Changes sync via normal git submodule commands

**Pattern**:
```bash
# Add domain skills when needed
git submodule add https://github.com/org/claude-marketing.git .claude/marketing

# Remove when not needed
git submodule deinit -f .claude/marketing
git rm -r .claude/marketing
```

---
