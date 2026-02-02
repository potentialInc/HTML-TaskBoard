---
name: test-run
description: Execute tests for a target project
---

# Test Run Skill

Executes E2E tests for the specified target and collects results.

> **⛔ CRITICAL: ALL STEPS ARE MANDATORY AND NON-SKIPPABLE ⛔**
>
> Every step in this document MUST be completed in order.
> - **DO NOT** skip any step
> - **DO NOT** assume any step is optional
> - **DO NOT** proceed to the next step until the current step is verified complete
>
> **SKIPPING ANY STEP = INCOMPLETE/FAILED TEST RUN**

---

## Input

- `target`: The project target (frontend, backend, frontend-dashboard)

---

## ⛔ Step 0: Collect Test Credentials (MANDATORY - DO NOT SKIP) ⛔

**IMPORTANT:** Before running tests, ALWAYS ask the user for test credentials using AskUserQuestion tool.

### Required Credentials by Target

| Target | Required User Types |
|--------|---------------------|
| frontend | PATIENT, COACH |
| frontend-dashboard | ADMIN |
| backend | All (ADMIN, COACH, PATIENT) |

### Prompt for Credentials

Use the AskUserQuestion tool to collect credentials from the user. Do NOT assume or hardcode any credentials.

**For frontend tests:**

Ask two questions:
1. "What are the PATIENT test credentials? (format: username/password)"
2. "What are the COACH test credentials? (format: username/password)"

**For frontend-dashboard tests:**

Ask one question:
1. "What are the ADMIN test credentials? (format: username/password)"

**For backend tests:**

Ask for all required credentials:
1. "What are the ADMIN test credentials? (format: username/password)"
2. "What are the COACH test credentials? (format: username/password)"
3. "What are the PATIENT test credentials? (format: username/password)"

### Update Test Fixtures with Collected Credentials

After collecting credentials, update the appropriate test fixture file:

| Target | Fixture File |
|--------|--------------|
| frontend | `frontend/test/fixtures/auth.fixture.ts` |
| frontend-dashboard | `frontend-dashboard/test/fixtures/auth.fixture.ts` |
| backend | `backend/test/fixtures/auth.fixture.ts` |

Update the credentials in the fixture before running tests:

```typescript
export const adminCredentials = {
  username: '{user_provided_username}',
  password: '{user_provided_password}',
  // ... other fields remain unchanged
};
```

---

## ⛔ MANDATORY SERVER GATE ⛔

> **🚨 CRITICAL: THIS IS A BLOCKING STEP 🚨**
>
> **YOU MUST NOT PROCEED TO STEP 4 (EXECUTE TESTS) UNTIL ALL REQUIRED SERVERS ARE RUNNING AND VERIFIED.**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - If you skip this step, tests **WILL fail** with timeout errors
>
> **FAILURE TO COMPLETE THIS STEP = 100% TEST FAILURE FOR FRONTEND TARGETS**

### Required Server Matrix

| Target | Backend (4000) | Frontend (5173) | Dashboard (5174) |
|--------|----------------|-----------------|------------------|
| backend | ❌ Not needed | ❌ Not needed | ❌ Not needed |
| frontend | ✅ **REQUIRED** | ✅ **REQUIRED** | ❌ Not needed |
| frontend-dashboard | ✅ **REQUIRED** | ❌ Not needed | ✅ **REQUIRED** |

### Mandatory Verification Sequence

**YOU MUST EXECUTE THIS COMMAND AND CHECK THE OUTPUT:**

```bash
echo "=== SERVER STATUS CHECK ===" && \
curl -s http://localhost:4000/api > /dev/null 2>&1 && echo "✅ Backend: RUNNING" || echo "❌ Backend: NOT RUNNING" && \
curl -s http://localhost:5173 > /dev/null 2>&1 && echo "✅ Frontend: RUNNING" || echo "❌ Frontend: NOT RUNNING" && \
curl -s http://localhost:5174 > /dev/null 2>&1 && echo "✅ Dashboard: RUNNING" || echo "❌ Dashboard: NOT RUNNING"
```

**After running this command:**

1. If ANY **required** server shows "❌ NOT RUNNING" → **STOP** and go to Step 2
2. After completing Step 2 → **RE-RUN this verification**
3. **ONLY proceed to Step 4 when ALL required servers show "✅ RUNNING"**

### ⚠️ DO NOT PROCEED CHECKLIST ⚠️

Before moving to Step 4, manually verify each checkbox:

- [ ] Backend server responding at http://localhost:4000/api (if required for target)
- [ ] Frontend server responding at http://localhost:5173 (if required for target)
- [ ] Dashboard server responding at http://localhost:5174 (if required for target)

**If any required checkbox is unchecked, STOP and complete Step 2 first.**

---

## ⛔ FALLBACK: Ask for Deployed URL (MANDATORY IF LOCAL FAILS) ⛔

> **🚨 CRITICAL: IF LOCAL SERVER STARTUP FAILS, YOU MUST ASK FOR URL 🚨**
>
> If Step 2 (server startup) fails after 2 attempts:
> 1. **DO NOT** skip the test
> 2. **DO NOT** give up
> 3. **MUST** ask the user for a deployed URL

### When to Use This Fallback

Use this IMMEDIATELY if:
- Local server fails to start after 2 attempts
- Server starts but crashes immediately
- Port is blocked or unavailable
- Any other local startup issue

### Ask for Deployed URL

Use **AskUserQuestion** tool:

```
Question: "로컬 서버 시작이 실패했습니다. 테스트를 실행할 배포된 URL을 제공해 주세요. (예: https://app.example.com)"
Header: "Deployed URL"
Options:
  1. "URL 제공" - I will provide a deployed URL
  2. "다시 시도" - Retry local server startup
```

### Update Playwright Config for Deployed URL

After receiving URL, update the playwright.config.ts:

```typescript
// The config should support BASE_URL environment variable
baseURL: process.env.BASE_URL || 'http://localhost:{port}',

// Skip local webServer when BASE_URL is provided
webServer: process.env.BASE_URL ? undefined : {
  command: 'npm run dev',
  // ...
},
```

### Run Tests with Deployed URL

```bash
cd {target} && BASE_URL={deployed_url} npx playwright test --project=chromium
```

### Required URLs by Target

| Target | URL Purpose | Example |
|--------|-------------|---------|
| frontend | Patient/Coach app | https://activitycoaching.example.com |
| frontend-dashboard | Admin dashboard | https://admin.example.com |
| backend | API server | (local only, no URL fallback) |

**⛔ DO NOT proceed without either local server OR deployed URL ⛔**

---

## ⛔ Step 1: Check Prerequisites (MANDATORY - DO NOT SKIP) ⛔

### For Backend Tests

No additional servers needed. Backend tests use test database.

Verify test setup:
```bash
cd backend && ls test/*.e2e-spec.ts
```

### For Frontend Tests

Required servers:
- Backend API: `http://localhost:4000`
- Frontend dev server: `http://localhost:5173`

Check servers:
```bash
curl -s http://localhost:4000/api > /dev/null && echo "Backend OK" || echo "Backend NOT running"
curl -s http://localhost:5173 > /dev/null && echo "Frontend OK" || echo "Frontend NOT running"
```

### For Frontend-Dashboard Tests

Required servers:
- Backend API: `http://localhost:4000`
- Dashboard dev server: `http://localhost:5174`

Check servers:
```bash
curl -s http://localhost:4000/api > /dev/null && echo "Backend OK" || echo "Backend NOT running"
curl -s http://localhost:5174 > /dev/null && echo "Dashboard OK" || echo "Dashboard NOT running"
```

---

## ⛔ Step 2: Auto-Start Missing Servers (MANDATORY - DO NOT SKIP) ⛔

> **🚨 CRITICAL: THIS IS A BLOCKING STEP 🚨**
>
> If ANY required server from the Server Gate check is "NOT RUNNING", you **MUST** complete this step.
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - **NEVER proceed to Step 4 without completing this step**
>
> **If server startup fails after 2 attempts → GO TO FALLBACK SECTION (Ask for Deployed URL)**
>
> Skipping this step = Test timeout failures = Wasted time

### Start Backend Server (if needed)

Check if backend is running:
```bash
curl -s http://localhost:4000/api > /dev/null 2>&1
```

If NOT running, start in background:
```bash
cd backend && npm run start:dev &
```

Wait for backend to be ready (max 30 seconds):
```bash
for i in {1..30}; do
  curl -s http://localhost:4000/api > /dev/null 2>&1 && break
  sleep 1
done
```

### Start Frontend Server (if needed)

Check if frontend is running:
```bash
curl -s http://localhost:5173 > /dev/null 2>&1
```

If NOT running, start in background:
```bash
cd frontend && npm run dev &
```

Wait for frontend to be ready (max 20 seconds):
```bash
for i in {1..20}; do
  curl -s http://localhost:5173 > /dev/null 2>&1 && break
  sleep 1
done
```

### Start Frontend-Dashboard Server (if needed)

Check if dashboard is running:
```bash
curl -s http://localhost:5174 > /dev/null 2>&1
```

If NOT running, start in background:
```bash
cd frontend-dashboard && npm run dev &
```

Wait for dashboard to be ready (max 20 seconds):
```bash
for i in {1..20}; do
  curl -s http://localhost:5174 > /dev/null 2>&1 && break
  sleep 1
done
```

### Server Startup Summary

After starting servers, display:
```markdown
## Servers Started

| Server | URL | Status |
|--------|-----|--------|
| Backend | http://localhost:4000 | Started |
| Frontend | http://localhost:5173 | Started |

Waiting for servers to be ready...
```

### If Server Fails to Start

If server doesn't respond after timeout:
```markdown
## Server Startup Failed

Could not start {server} after 30 seconds.

Possible issues:
- Port already in use
- Dependencies not installed (run `npm install`)
- Database not running (for backend)

Check logs and try again.
```

> **⛔ MANDATORY: IF SERVER FAILS AFTER 2 ATTEMPTS ⛔**
>
> If you cannot start the local server after 2 attempts:
> 1. **DO NOT** skip the test
> 2. **DO NOT** give up
> 3. **MUST** go to the **FALLBACK** section above
> 4. **MUST** ask user for deployed URL
> 5. **MUST** run tests against deployed URL
>
> **NEVER complete a test run without either local OR deployed URL working**

### ⛔ FINAL VERIFICATION (MANDATORY) ⛔

**After starting all required servers, you MUST run this final check:**

```bash
echo "=== FINAL SERVER VERIFICATION ===" && \
curl -s http://localhost:4000/api > /dev/null 2>&1 && echo "✅ Backend: READY" || echo "❌ Backend: FAILED" && \
curl -s http://localhost:5173 > /dev/null 2>&1 && echo "✅ Frontend: READY" || echo "❌ Frontend: FAILED" && \
curl -s http://localhost:5174 > /dev/null 2>&1 && echo "✅ Dashboard: READY" || echo "❌ Dashboard: FAILED"
```

**⛔ If ANY required server shows "❌ FAILED":**
1. Check the log file (`/tmp/backend.log`, `/tmp/frontend.log`, `/tmp/dashboard.log`)
2. Fix the issue
3. Restart the server
4. **Re-run this verification**

**DO NOT proceed to Step 4 until ALL required servers show "✅ READY"**

---

## ⛔ Step 3: Install Playwright Browsers (MANDATORY - DO NOT SKIP) ⛔

> **This step is NOT optional. You MUST verify Playwright is installed before running tests.**

For frontend tests, check Playwright is ready:

```bash
cd {target} && npx playwright --version
```

If Playwright browsers not installed:
```bash
cd {target} && npx playwright install
```

---

## ⛔ Step 4: Execute Tests (MANDATORY - DO NOT SKIP) ⛔

> **🚨 PRE-EXECUTION GATE 🚨**
>
> **Before running ANY test command, verify:**
> 1. You completed the MANDATORY SERVER GATE check
> 2. You completed Step 2 (if servers were not running) OR have a deployed URL
> 3. All required servers are verified running (FINAL VERIFICATION passed) OR using deployed URL
>
> **If you skipped any of the above steps, STOP NOW and go back.**
>
> **Quick verification before proceeding (for local servers):**
> ```bash
> # Run this BEFORE executing tests
> curl -s http://localhost:4000/api > /dev/null && echo "Backend: OK" || echo "Backend: FAIL - STOP!"
> curl -s http://localhost:5173 > /dev/null && echo "Frontend: OK" || echo "Frontend: FAIL - STOP!"
> ```
> **If ANY required server shows "FAIL" and you DON'T have a deployed URL, go back to Step 2 or FALLBACK section.**
>
> **For deployed URL testing:**
> ```bash
> BASE_URL={deployed_url} npx playwright test --project=chromium
> ```

### Backend (Jest)

```bash
cd backend && npm run test:e2e -- --json --outputFile=test-results/results.json 2>&1
```

Capture:
- Exit code
- JSON results file
- Console output

### Frontend (Playwright)

```bash
cd frontend && npx playwright test --reporter=json > test-results/results.json 2>&1
```

Or with npm script:
```bash
cd frontend && npm run test:e2e -- --reporter=json
```

Capture:
- Exit code
- JSON results file
- Console output

### Frontend-Dashboard (Playwright)

```bash
cd frontend-dashboard && npx playwright test --reporter=json > test-results/results.json 2>&1
```

---

## ⛔ Step 5: Parse Test Results (MANDATORY - DO NOT SKIP) ⛔

> **🚨 CRITICAL: THIS STEP IS ABSOLUTELY MANDATORY 🚨**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - **SKIPPING THIS STEP = TEST RUN FAILURE**
>
> You MUST parse the raw test output and transform it into the structured format defined in Step 6.
> - NEVER save raw Playwright/Jest JSON output directly
> - NEVER save console log output as JSON
> - ALWAYS transform raw output into the structured format with `target`, `timestamp`, `summary`, `byCategory`, `byErrorType`, `failedTests`, and `recommendations` fields
>
> If you skip this step, the test results will be unusable and inconsistent across targets.

### Raw Output Formats (for parsing)

#### Jest Results Format (Backend)

```json
{
  "numTotalTests": 103,
  "numPassedTests": 100,
  "numFailedTests": 2,
  "numPendingTests": 1,
  "testResults": [
    {
      "name": "test/auth.e2e-spec.ts",
      "assertionResults": [
        {
          "title": "should login with valid credentials",
          "status": "passed",
          "duration": 150
        }
      ]
    }
  ]
}
```

### Playwright Results Format (Frontend)

```json
{
  "suites": [
    {
      "title": "Login Page",
      "file": "tests/auth/login.spec.ts",
      "specs": [
        {
          "title": "should display login form",
          "ok": true,
          "tests": [
            {
              "status": "passed",
              "duration": 2500
            }
          ]
        }
      ]
    }
  ],
  "stats": {
    "total": 76,
    "passed": 74,
    "failed": 2,
    "skipped": 0
  }
}
```

### Error Type Classification

Automatically classify errors by pattern matching:

| Error Pattern | Error Type | Severity | Common Cause |
|---------------|------------|----------|--------------|
| `not visible`, `not found`, `Locator` | UI element not visible | medium | Selector changed or element hidden |
| `Timeout`, `exceeded` | Timeout exceeded | medium | Slow response or missing element |
| `No exercises`, `No data`, `empty` | No data available | high | Test data setup issue |
| `Layout`, `viewport`, `mobile` | Layout mismatch | low | Responsive CSS differences |
| `keyboard`, `focus`, `tabIndex`, `aria` | Accessibility issue | medium | Missing a11y attributes |
| `Navigation`, `redirect`, `URL` | Navigation failed | medium | Route guard or navigation issue |
| `Count`, `mismatch` | Count mismatch | medium | Data count discrepancy |

### Category Extraction

Extract category from test file path:

```
tests/auth/login.spec.ts     → auth
tests/coach/calendar.spec.ts → coach
tests/patient/home.spec.ts   → patient
```

---

## ⛔ Step 6: Collect Results (MANDATORY - DO NOT SKIP) ⛔

> **🚨 CRITICAL: THIS STEP IS ABSOLUTELY MANDATORY 🚨**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - **SKIPPING THIS STEP = TEST RUN FAILURE**
>
> This is the ONLY format that should be saved to `{TARGET_PREFIX}_TEST_RESULTS.json`.
> You must transform the raw test output from Step 5 into this exact structure.

Create enhanced results object with analysis:

```json
{
  "target": "{target}",
  "timestamp": "{ISO timestamp}",
  "summary": {
    "total": 594,
    "passed": 440,
    "failed": 154,
    "skipped": 0,
    "passRate": 74.07,
    "duration": 125000
  },
  "byCategory": {
    "auth": { "total": 56, "passed": 52, "failed": 4, "passRate": 92.9 },
    "coach": { "total": 120, "passed": 98, "failed": 22, "passRate": 81.7 },
    "patient": { "total": 418, "passed": 290, "failed": 128, "passRate": 69.4 }
  },
  "byErrorType": {
    "No data available": {
      "count": 28,
      "severity": "high",
      "cause": "Test data setup issue - patient has no exercises or sessions",
      "fix": "Update seed data to include required test data",
      "affectedFiles": ["exercise-detail.spec.ts", "exercise.spec.ts", "calendar.spec.ts"]
    },
    "UI element not visible": {
      "count": 12,
      "severity": "medium",
      "cause": "Element selector changed or conditional rendering",
      "fix": "Update test selectors or check component visibility state",
      "affectedFiles": ["login.spec.ts", "chat.spec.ts", "home.spec.ts"]
    },
    "Accessibility issue": {
      "count": 10,
      "severity": "medium",
      "cause": "Missing keyboard navigation or focus management",
      "fix": "Add tabIndex, aria-labels, or focus handlers to components",
      "affectedFiles": ["home.spec.ts", "exercise.spec.ts", "chat.spec.ts"]
    },
    "Layout mismatch": {
      "count": 8,
      "severity": "low",
      "cause": "Responsive design differences in mobile/tablet viewports",
      "fix": "Review viewport-specific CSS and media queries",
      "affectedFiles": ["calendar.spec.ts", "chat.spec.ts", "exercise.spec.ts"]
    }
  },
  "failedTests": [
    {
      "file": "test/tests/auth/login.spec.ts",
      "line": 107,
      "title": "should show patient demo credentials (mk/123)",
      "category": "auth",
      "errorType": "UI element not visible",
      "error": "Locator('text=mk/123') not visible within 5000ms",
      "duration": 5234
    }
  ],
  "recommendations": [
    {
      "priority": 1,
      "action": "Fix test data seeding",
      "errorType": "No data available",
      "impact": "28 tests (18%)",
      "effort": "low",
      "files": ["backend/src/database/seeders/user.seed.ts"]
    },
    {
      "priority": 2,
      "action": "Update UI selectors",
      "errorType": "UI element not visible",
      "impact": "12 tests (8%)",
      "effort": "medium",
      "files": ["Multiple test files"]
    },
    {
      "priority": 3,
      "action": "Add accessibility attributes",
      "errorType": "Accessibility issue",
      "impact": "10 tests (6%)",
      "effort": "medium",
      "files": ["Multiple component files"]
    }
  ]
}
```

---

## ⛔ Step 7: Save Results (MANDATORY - DO NOT SKIP) ⛔

> **🚨 CRITICAL: THIS STEP IS ABSOLUTELY MANDATORY 🚨**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - **YOU MUST CREATE ALL 3 FILES LISTED BELOW**
> - **SKIPPING THIS STEP = TEST RUN FAILURE**

> **VALIDATION CHECK BEFORE SAVING**
>
> Before saving, verify the JSON has these required top-level fields:
> - `target` (string)
> - `timestamp` (ISO string)
> - `summary` (object with total, passed, failed, skipped, passRate)
> - `byCategory` (object)
> - `byErrorType` (object)
> - `failedTests` (array)
> - `recommendations` (array)
>
> If the JSON starts with `config`, `suites`, `numTotalTests`, or console logs, **STOP** - you skipped Step 5-6.

### Dynamic File Naming

Convert target name to uppercase prefix for file names:
- Replace hyphens `-` with underscores `_`
- Convert to UPPERCASE

Examples:
| Target | Prefix | Files |
|--------|--------|-------|
| frontend | FRONTEND | `FRONTEND_TEST_RESULTS.json`, `FRONTEND_TESTCASE_STATUS.md` |
| frontend-dashboard | FRONTEND_DASHBOARD | `FRONTEND_DASHBOARD_TEST_RESULTS.json`, `FRONTEND_DASHBOARD_TESTCASE_STATUS.md` |
| backend | BACKEND | `BACKEND_TEST_RESULTS.json`, `BACKEND_TESTCASE_STATUS.md` |
| coach-dashboard | COACH_DASHBOARD | `COACH_DASHBOARD_TEST_RESULTS.json`, `COACH_DASHBOARD_TESTCASE_STATUS.md` |

### Save Files

Save results file for update skill:

```
.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.json
```

Also save human-readable report:

```
.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.md
```

Also save/update test case status:

```
.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md
```

### TEST_RESULTS.md Format

Generate a comprehensive markdown report with the following structure:

```markdown
# Test Results: {target}

**Executed:** {date} {time} KST
**Duration:** {minutes}m {seconds}s
**Pass Rate:** {passRate}% ({passed}/{total})

---

## Summary by Category

| Category | Total | Passed | Failed | Pass Rate |
|----------|-------|--------|--------|-----------|
| Auth     | 56    | 52     | 4      | 92.9%     |
| Coach    | 120   | 98     | 22     | 81.7%     |
| Patient  | 418   | 290    | 128    | 69.4%     |
| **Total**| 594   | 440    | 154    | 74.1%     |

---

## Failure Analysis

### High Priority ({count} tests)

#### {errorType} ({count} tests)
**Root Cause:** {cause}
**Fix:** {fix}

Affected files:
- `{file1}` ({count} tests)
- `{file2}` ({count} tests)

### Medium Priority ({count} tests)

#### {errorType} ({count} tests)
**Root Cause:** {cause}
**Fix:** {fix}

### Low Priority ({count} tests)

#### {errorType} ({count} tests)
**Root Cause:** {cause}
**Fix:** {fix}

---

## Recommended Actions

| Priority | Action | Impact | Effort |
|----------|--------|--------|--------|
| 1 | {action} | {count} tests ({percent}%) | {effort} |
| 2 | {action} | {count} tests ({percent}%) | {effort} |
| 3 | {action} | {count} tests ({percent}%) | {effort} |

---

## Full Failed Test List

<details>
<summary>Auth ({count} failures)</summary>

| Test | Error Type | File |
|------|------------|------|
| {title} | {errorType} | `{file}:{line}` |

</details>

<details>
<summary>Coach ({count} failures)</summary>

| Test | Error Type | File |
|------|------------|------|
| {title} | {errorType} | `{file}:{line}` |

</details>

<details>
<summary>Patient ({count} failures)</summary>

| Test | Error Type | File |
|------|------------|------|
| {title} | {errorType} | `{file}:{line}` |

</details>

---

## Artifacts

- JSON Results: `.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.json`
- Test Status: `.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md`
- Playwright HTML: `{target}/test/reports/html/index.html`
```

---

## ⛔ MANDATORY RESULT FILES VERIFICATION ⛔

> **🚨 CRITICAL: THIS IS A BLOCKING STEP 🚨**
>
> **YOU MUST NOT COMPLETE THE TEST RUN WITHOUT GENERATING ALL RESULT FILES.**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - If you skip this step, test results will be **LOST** and **NOT DOCUMENTED**
>
> **FAILURE TO COMPLETE THIS STEP = INCOMPLETE TEST RUN**

### Required Files Checklist

Before proceeding to Step 8, verify you have created ALL of these files:

| File | Path | Created? |
|------|------|----------|
| JSON Results | `.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.json` | [ ] |
| MD Report | `.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.md` | [ ] |
| Status Update | `.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md` | [ ] |

### Verification Command

**Run this to verify files exist:**

```bash
ls -la .claude-project/status/{target}/*.json .claude-project/status/{target}/*.md
```

**Expected output:** All 3 files should be listed for the target.

### ⚠️ DO NOT PROCEED CHECKLIST ⚠️

Before moving to Step 8:

- [ ] `{TARGET_PREFIX}_TEST_RESULTS.json` exists and contains structured data
- [ ] `{TARGET_PREFIX}_TEST_RESULTS.md` exists with human-readable report
- [ ] `{TARGET_PREFIX}_TESTCASE_STATUS.md` is updated with test run info

**If ANY file is missing:**
1. Go back to Step 5-7
2. Create the missing file(s)
3. Re-run this verification

**⛔ DO NOT SAY "Test run complete" UNTIL ALL FILES ARE GENERATED ⛔**

---

## ⛔ Step 8: Display Results (MANDATORY - DO NOT SKIP) ⛔

> **🚨 THIS STEP IS MANDATORY 🚨**
>
> - This is **NOT optional**
> - This is **NOT skippable**
> - You MUST display a summary to the user

Display a concise summary in the console:

```markdown
## Test Results: {target}

**Executed:** {timestamp}
**Duration:** {duration}s
**Pass Rate:** {passRate}%

---

### Quick Summary

| Metric | Value |
|--------|-------|
| Total | {total} |
| Passed | {passed} |
| Failed | {failed} |

---

### Category Breakdown

| Category | Pass Rate | Failed |
|----------|-----------|--------|
| Auth | {rate}% | {count} |
| Coach | {rate}% | {count} |
| Patient | {rate}% | {count} |

---

### Top Issues (by frequency)

| Error Type | Count | Severity | Suggested Fix |
|------------|-------|----------|---------------|
| {errorType} | {count} | {severity} | {fix} |
| {errorType} | {count} | {severity} | {fix} |
| {errorType} | {count} | {severity} | {fix} |

---

### Recommended Next Steps

1. **{action}** - {impact} ({effort} effort)
2. **{action}** - {impact} ({effort} effort)
3. **{action}** - {impact} ({effort} effort)

---

### Artifacts

- Full Report: `.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.md`
- JSON Data: `.claude-project/status/{target}/{TARGET_PREFIX}_TEST_RESULTS.json`
- Test Status: `.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md`
- HTML Report: `{target}/test/reports/html/index.html`
```

---

## Output

Returns:
- Test execution results (pass/fail counts)
- List of failed tests with errors
- Results files for update skill

> **⛔ FINAL COMPLETION CHECK ⛔**
>
> **BEFORE saying the test run is complete, verify:**
>
> 1. All 3 result files exist for EACH target tested:
>    - `{TARGET_PREFIX}_TEST_RESULTS.json`
>    - `{TARGET_PREFIX}_TEST_RESULTS.md`
>    - `{TARGET_PREFIX}_TESTCASE_STATUS.md`
>
> 2. Each JSON file contains structured data (not raw console output)
>
> 3. Each MD file contains readable summary
>
> **If ANY file is missing, DO NOT complete the test run. Go back and create it.**

---

## Running Specific Tests

The skill can accept optional filters:

```bash
# Run specific file
/test frontend --run --file=login.spec.ts

# Run specific test
/test frontend --run --grep="should display"
```

Translates to:
```bash
npx playwright test tests/auth/login.spec.ts
npx playwright test --grep "should display"
```

---

## Retry Failed Tests

If tests fail, offer to retry:

```
Question: "2 tests failed. Retry failed tests only?"
Header: "Retry"
Options:
  1. "Yes" - Retry failed tests
  2. "No" - Continue with results
```

Retry command:
```bash
npx playwright test --last-failed
```

---

## Error Handling

| Error | Action |
|-------|--------|
| Server not running | Show start instructions, ask to wait |
| Playwright not installed | Run `npx playwright install` |
| Test timeout | Report as failed, include timeout info |
| All tests fail | Check server connectivity, report issue |
| Results file missing | Parse from console output instead |

---

## Performance Tips

### Run tests in parallel
```bash
npx playwright test --workers=4
```

### Run specific category
```bash
npx playwright test tests/auth/
```

### Headed mode for debugging
```bash
npx playwright test --headed
```
