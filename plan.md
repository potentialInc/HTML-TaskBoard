# Plan: Remove Redundant `skills/generate-ppt` Directory

## Analysis Summary

After comparing the two files, I found **significant content duplication**:

### What We Have

| Location | File | Purpose |
|----------|------|---------|
| `commands/generate-ppt.md` | Command | **Complete implementation** - workflow, parsing, templates, brand guidelines, HTML structure |
| `skills/generate-ppt/SKILL.md` | Skill | **Reference documentation** - same content plus additional themes and slide types |

### Content Overlap

Both files contain:
- Same brand colors table (#624DFF, #050042, #1e293b, etc.)
- Same typography specification (Inter font)
- Same logo SVG (identical 200x88 inline SVG)
- Same HTML template structure
- Same keyboard shortcuts
- Same output location (`presentations/`)

### Differences

| Feature | Command | Skill |
|---------|---------|-------|
| Two-step workflow (topic→md→html) | ✅ | ❌ |
| Additional themes (Dark, Minimal) | ❌ | ✅ |
| Additional slide types (Two-Column, Code, Image) | ❌ | ✅ |
| Best practices section | ❌ | ✅ |

## Recommendation

**Delete `skills/generate-ppt/` directory** - it's largely redundant.

The command file (`commands/generate-ppt.md`) is the authoritative implementation and already contains everything needed to generate presentations. The skill file mostly duplicates this content.

### What to Preserve

The skill has a few useful additions that could optionally be merged into the command:
1. Dark and Minimal theme CSS variables
2. Two-column, code, and image slide type templates
3. Best practices guidelines

However, these are **nice-to-haves** rather than essential. The command works without them.

## Proposed Action

1. **Delete** `skills/generate-ppt/` directory entirely
2. (Optional) Add Dark/Minimal themes to command if desired later

## Files to Remove

```
skills/generate-ppt/
└── SKILL.md
```

---

**Awaiting approval to proceed.**
