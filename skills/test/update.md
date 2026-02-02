---
name: test-update
description: Update {TARGET_PREFIX}_TESTCASE_STATUS.md with test execution results
---

# Test Update Skill

Updates `{TARGET_PREFIX}_TESTCASE_STATUS.md` with the results from test execution.

**File Naming Convention:**
- Convert target name to uppercase prefix
- Replace hyphens `-` with underscores `_`

| Target | Prefix |
|--------|--------|
| backend | BACKEND |
| frontend | FRONTEND |
| frontend-dashboard | FRONTEND_DASHBOARD |

---

## Input

- `target`: The project target (frontend, backend, frontend-dashboard)
- Test results from `run.md` skill

---

## Step 1: Read Test Results

Read the results file:
```
.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.json
```

Example: `.claude-project/status/backend/BACKEND_TEST_RESULTS.json`

If file doesn't exist, return error: "No test results found. Run tests first."

---

## Step 2: Read Current TESTCASE_STATUS.md

Read the status file:
```
.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md
```

Example: `.claude-project/status/backend/BACKEND_TESTCASE_STATUS.md`

Parse markdown tables to extract current status for each test case.

---

## Step 3: Match Results to Test Cases

For each test result:

1. **Find matching entry in TESTCASE_STATUS.md**

   Match by:
   - Test title (exact or fuzzy match)
   - File path + line number
   - URL pattern

2. **Handle unmatched results**

   If test result has no matching STATUS entry:
   - Add to "orphan tests" list
   - Optionally add new row to STATUS

---

## Step 4: Update Status Values

### Status Mapping

| Test Result | New Status |
|-------------|------------|
| passed | Complete |
| failed | In Progress |
| skipped | Skipped |
| pending | Not Started |

### Update Columns

For each matched test case, update:

| Column | New Value |
|--------|-----------|
| Status | Based on result (see mapping) |
| Last Tested | Today's date (YYYY-MM-DD) |
| Memo | Add error message if failed |

---

## Step 5: Preserve User Data

**IMPORTANT:** Do not overwrite user-maintained data:

### Preserve Memo Column

If Memo contains user notes:
- Append error message, don't replace
- Keep `[Manual]` tagged notes
- Format: `{user_note} | Error: {error_message}`

### Preserve Manual Status Overrides

If Memo contains `[Manual]`:
- Do NOT update Status column
- This indicates user manually set status

Example:
```
| Title | Status | Memo |
|-------|--------|------|
| test case | Blocked | [Manual] Waiting for API fix |
```
This row's Status will not be updated.

---

## Step 6: Handle Failed Tests

For failed tests, add error info to Memo:

**Before:**
```
| Title | Status | Last Tested | Memo |
|-------|--------|-------------|------|
| should redirect coach | Complete | 2026-01-20 | |
```

**After:**
```
| Title | Status | Last Tested | Memo |
|-------|--------|-------------|------|
| should redirect coach | In Progress | 2026-01-29 | Error: Expected /coach/, got /login |
```

### Truncate Long Errors

If error message > 100 characters:
- Truncate with "..."
- Full error in TEST_RESULTS.md

---

## Step 7: Update Summary Table

Recalculate the summary table at the bottom of TESTCASE_STATUS.md:

```markdown
## Summary

| Category | Total | Complete | In Progress | Not Started | Skipped | Blocked |
|----------|-------|----------|-------------|-------------|---------|---------|
| Auth - Login | 18 | 16 | 2 | 0 | 0 | 0 |
| Patient - Home | 24 | 24 | 0 | 0 | 0 | 0 |
| **Total** | 76 | 72 | 2 | 0 | 2 | 0 |
```

Count by Status value for each category.

---

## Step 8: Update Metadata

Update the header metadata:

```markdown
# Test Case Status: {App Name}

> **Application:** {target}
> **Test Framework:** {Jest/Playwright}
> **Last Generated:** {original_date}
> **Last Tested:** {today's date}      ← ADD/UPDATE this line
> **Total Test Cases:** {count}
> **Pass Rate:** {pass_rate}%          ← ADD/UPDATE this line
```

---

## Step 9: Write Updated File

Write the updated status file:
```
.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md
```

Example: `.claude-project/status/backend/BACKEND_TESTCASE_STATUS.md`

---

## Step 10: Generate Update Report

```markdown
## Status Update: {target}

**Updated at:** {timestamp}

---

### Changes Summary

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Complete | 70 | 72 | +2 |
| In Progress | 4 | 2 | -2 |
| Not Started | 2 | 2 | 0 |
| Skipped | 0 | 0 | 0 |
| Pass Rate | 92% | 95% | +3% |

---

### Status Changes

| Test Case | Old Status | New Status | Notes |
|-----------|------------|------------|-------|
| should redirect coach | Complete | In Progress | Failed: Expected /coach/ |
| should display greeting | In Progress | Complete | Now passing |

---

### Newly Tested

Tests that were run for the first time:

| Test Case | Result | File |
|-----------|--------|------|
| should validate email | Complete | `signup.spec.ts:45` |

---

### Still Failing

Tests that remain in "In Progress" status:

| Test Case | File | Error |
|-----------|------|-------|
| should handle timeout | `api.spec.ts:120` | Timeout exceeded |

---

### Updated File

`.claude-project/status/{target}/TESTCASE_STATUS.md`
```

---

## Output

Returns:
- Updated TESTCASE_STATUS.md
- Summary of changes
- List of still-failing tests

---

## Merge Strategies

### New Test Results + Existing STATUS

```
For each test in STATUS:
  If has matching result:
    Update status, date, memo
  Else:
    Keep existing values (test wasn't run)
```

### Orphan Results (not in STATUS)

```
For each result without STATUS entry:
  Option 1: Add new row to STATUS (recommended)
  Option 2: Report as orphan, don't add
```

Use **AskUserQuestion** if orphans found:

```
Question: "Found 3 tests not in TESTCASE_STATUS.md. Add them?"
Header: "New Tests"
Options:
  1. "Yes" - Add new rows for these tests
  2. "No" - Skip, only update existing entries
```

---

## Error Handling

| Error | Action |
|-------|--------|
| No results file | Return error, suggest running tests first |
| STATUS file missing | Return error, suggest running /generate-testcase |
| No matching entries | Report orphan tests, offer to add |
| Parse error | Log warning, skip problematic entries |

---

## Rollback Support

Before writing, backup current STATUS:
```
.claude-project/status/{target}/TESTCASE_STATUS.md.backup
```

If update fails, restore from backup.
