---
description: Generate test case documentation for backend, frontend, and frontend-dashboard with test code mapping
argument-hint: Optional target (e.g., backend, frontend-dashboard, frontend, all). Default: all
---

You are a test case documentation generator. Your task is to create comprehensive test case status documents for **three locations**: backend (API tests), frontend (E2E tests), and frontend-dashboard (Admin E2E tests), including automatic mapping of existing test code files.

---

## Workflow Overview

1. Parse arguments and determine targets
2. Read project documentation
3. Scan existing test files for each target
3.5. **Auto-detect Critical Scenarios** (roles, panels, CRUD resources)
4. Generate/update test case status documents (including auto-detected tests)
5. Report results

---

## Step 1: Parse Arguments

Read `$ARGUMENTS` to determine the target:

| Argument | Action |
|----------|--------|
| Empty or `all` | Process all three: backend, frontend, frontend-dashboard |
| `backend` | Process backend only |
| `frontend` | Process frontend only |
| `frontend-dashboard` | Process frontend-dashboard only |

---

## Step 2: Read Reference Documentation

Read the following documentation files for context:

| File | Purpose |
|------|---------|
| `.claude-project/docs/PROJECT_API.md` | API endpoints reference |
| `.claude-project/docs/PROJECT_DATABASE.md` | Database schema |
| `.claude-project/docs/PROJECT_KNOWLEDGE.md` | Project knowledge |
| `.claude-project/docs/PROJECT_API_INTEGRATION.md` | Integration details |

**Optional**: If backend is running, fetch Swagger at `http://localhost:4000/api` for live API documentation.

---

## Step 2.5: Collect Test Configuration (CRITICAL)

**Collect test configuration - ask user for credentials, auto-read the rest:**

### Ask User for Credentials

Use **AskUserQuestion** to collect test credentials:

```
Question: "Please provide test credentials (e.g., admin/12341234)"
```

Collect for each role:
- ADMIN credentials (ID/password)
- COACH credentials (if applicable)
- PATIENT/USER credentials (if applicable)

### Auto-read Login Field

Read from `backend/src/modules/auth/dtos/login.dto.ts`:
- Check if DTO uses `username` or `email`

### Auto-read JWT Secret

Read from `backend/.env`:
- Look for `JWT_SECRET`, `AUTH_JWT_SECRET`, or similar

### Auto-read Token Path

Read from `backend/src/modules/auth/auth.service.ts`:
- Find return statement to determine token path (e.g., `data.token`)

### Rules
1. **Ask user** for credentials only
2. **Auto-read** login field, JWT secret, token path from code
3. Test fixtures MUST use collected values
4. NEVER hardcode project-specific values

---

## Step 3: Scan Test Files (Per Target)

### 3.1 Backend Test Structure

**Test Framework**: Jest (NestJS)

**Locations**:
- Unit tests: `backend/src/**/*.spec.ts`
- E2E tests: `backend/test/**/*.e2e-spec.ts`

**Parsing Patterns** (Jest):
| Pattern | Captures |
|---------|----------|
| `describe\(['"](.+?)['"]` | Test suite/category name |
| `it\(['"](.+?)['"]` | Individual test case |
| `it\.skip\(['"](.+?)['"]` | Skipped test |

**URL Extraction**:
- From describe block: `describe('POST /auth/login'` → URL: `/auth/login`, Method: `POST`
- Pattern: `(GET|POST|PATCH|PUT|DELETE)\s+(/[^\s'"]+)`

**User Type Mapping for Backend**:
| URL Pattern | User Type |
|-------------|-----------|
| `/auth/login`, `/auth/register` | Public |
| `/admin/*` | ADMIN |
| `/meetings/coach/*` | COACH |
| `/meetings/patient/*` | PATIENT |
| `/exercises` (GET) | Requires Auth |
| `/users` (Admin endpoints) | ADMIN |

### 3.2 Frontend Test Structure

**Test Framework**: Playwright

**Locations**:
- Spec files: `frontend/test/tests/**/*.spec.ts`
- Page objects: `frontend/test/pages/**/*.page.ts`

**Parsing Patterns** (Playwright):
| Pattern | Captures |
|---------|----------|
| `test\.describe\(['"](.+?)['"]` | Test suite/category |
| `test\(['"](.+?)['"]` | Individual test case |
| `test\.skip\(['"](.+?)['"]` | Skipped test |

**URL Extraction**:
- From page objects: `readonly url = '/path'`
- Link via import: `import { CoachSignupPage } from '../../pages/auth/coach-signup.page'`

**User Type Mapping for Frontend**:
| Path Pattern | User Type |
|--------------|-----------|
| `tests/auth/login*` | Public |
| `tests/auth/signup*` | Public |
| `tests/coach/*` | COACH |
| `tests/patient/*` | PATIENT |

### 3.3 Frontend-Dashboard Test Structure

**Test Framework**: Playwright

**Locations**:
- Spec files: `frontend-dashboard/test/tests/**/*.spec.ts`
- Page objects: `frontend-dashboard/test/pages/**/*.page.ts` (if exists)

**User Type**: All tests are for **ADMIN** users (dashboard)

---

## Step 3.5: Auto-detect Critical Scenarios (Project Independent)

**System Analysis → Auto-generate Test Cases**

This step automatically detects system components and generates critical test cases that should exist regardless of project specifics. These tests catch common issues like broken signup, token leakage, and CRUD failures.

### 3.5.1 Detect System Roles

**Scan for roles enum:**
```
backend/src/shared/enums/roles.enum.ts
backend/src/common/enums/roles.enum.ts
backend/src/enums/roles.enum.ts
```

**Extract pattern:**
```typescript
export enum Roles {
  ADMIN = 'admin',
  COACH = 'coach',
  PATIENT = 'patient',
}
```

**Store as:** `detected_roles[]` (e.g., ['ADMIN', 'COACH', 'PATIENT'])

### 3.5.2 Detect Frontend Panels

**Scan project structure for multiple frontend apps:**

| Directory | Panel Type | Default Role |
|-----------|------------|--------------|
| `frontend/` | Patient/User App | PATIENT |
| `frontend-dashboard/` | Admin Dashboard | ADMIN |
| `frontend-coach/` | Coach App (if exists) | COACH |
| `frontend-admin/` | Admin App (if exists) | ADMIN |

**Store as:** `detected_panels[]` (e.g., ['frontend', 'frontend-dashboard'])

### 3.5.3 Detect CRUD Resources (Backend)

**Scan controllers:**
```
backend/src/modules/*/*.controller.ts
backend/src/modules/*/controllers/*.controller.ts
```

**Extract from decorators:**

| Decorator | Information |
|-----------|-------------|
| `@Controller('path')` | Resource base path |
| `@Post()`, `@Get()`, `@Patch()`, `@Delete()` | CRUD operations |
| `@Roles(...)` or `@AdminOnly` | Required role |
| `@Public()` | No auth required |

**Store as:** `detected_resources[]`
```
{
  name: 'exercises',
  basePath: '/exercises',
  operations: ['create', 'findAll', 'findOne', 'update', 'remove'],
  requiredRole: 'ADMIN'
}
```

### 3.5.4 Generate Role-based Test Cases

For EACH detected role, add these test cases to TESTCASE_STATUS.md:

**Backend Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `{role} should signup successfully` | `POST /auth/register` | Auth - Signup | Public |
| `{role} should login with valid credentials` | `POST /auth/login` | Auth - Login | Public |
| `{role} should access own profile` | `GET /users/me` | Users - Profile | {ROLE} |
| `{role} should NOT access admin endpoints` | `GET /admin/*` | Auth - Authorization | {ROLE} |

**Frontend Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `{role} signup flow should complete` | `/signup/{role}` | Auth - Signup | Public |
| `{role} login should redirect to dashboard` | `/login` | Auth - Login | Public |
| `{role} should see correct navigation` | `/{role}/home` | Navigation | {ROLE} |

### 3.5.5 Generate Session/Token Test Cases

For EACH detected panel combination:

**Token Isolation Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `{panelA} token should NOT work in {panelB}` | `/api/*` | Auth - Token Isolation | Multiple |
| `{role} token should be rejected by {other_role} panel` | `/login` | Auth - Cross-panel | Multiple |

**Session Management Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `logout should clear session completely` | `POST /auth/logout` | Auth - Logout | {ROLE} |
| `page reload after logout should redirect to login` | `/` | Auth - Session | Public |
| `expired token should return 401` | `/users/me` | Auth - Token Expiry | {ROLE} |
| `invalid token format should return 401` | `/users/me` | Auth - Token Validation | Public |

### 3.5.6 Generate CRUD Flow Test Cases

For EACH detected resource with CRUD operations:

**Backend CRUD Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `should create {resource}` | `POST /{resource}` | {Resource} - Create | {requiredRole} |
| `should list all {resource}` | `GET /{resource}` | {Resource} - Read | {requiredRole} |
| `should get {resource} by id` | `GET /{resource}/:id` | {Resource} - Read | {requiredRole} |
| `should update {resource}` | `PATCH /{resource}/:id` | {Resource} - Update | {requiredRole} |
| `should delete {resource}` | `DELETE /{resource}/:id` | {Resource} - Delete | {requiredRole} |
| `should complete full CRUD lifecycle` | `/{resource}` | {Resource} - Lifecycle | {requiredRole} |

**Validation Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `should reject invalid {resource} data` | `POST /{resource}` | {Resource} - Validation | {requiredRole} |
| `should return 404 for non-existent {resource}` | `GET /{resource}/:id` | {Resource} - Error | {requiredRole} |

### 3.5.7 Generate Authorization Matrix Test Cases

Build role × endpoint matrix:

```
For each endpoint with @Roles decorator:
  For each role in detected_roles:
    If role in allowed_roles:
      Add: "{role} should access {endpoint}" → Status: Not Started
    Else:
      Add: "{role} should be denied {endpoint}" → Status: Not Started
```

**Example Generated Tests:**

| Test Title | URL | Category | User Type |
|------------|-----|----------|-----------|
| `ADMIN should access GET /admin/dashboard` | `GET /admin/dashboard` | Admin - Access | ADMIN |
| `COACH should be denied GET /admin/dashboard` | `GET /admin/dashboard` | Admin - Denied | COACH |
| `PATIENT should be denied GET /admin/dashboard` | `GET /admin/dashboard` | Admin - Denied | PATIENT |

### 3.5.8 Critical Scenario Categories

All auto-generated tests should be grouped under these categories in TESTCASE_STATUS.md:

| Category | Description |
|----------|-------------|
| Auth - Signup | User registration flows |
| Auth - Login | Login and token generation |
| Auth - Logout | Session cleanup |
| Auth - Token Isolation | Cross-panel token rejection |
| Auth - Authorization | Role-based access control |
| {Resource} - CRUD | Resource lifecycle tests |
| {Resource} - Validation | Input validation tests |

### 3.5.9 Deduplication Rules

Before adding auto-detected test cases:

1. **Check existing tests** - Don't duplicate if similar test exists
2. **Match by URL + Method** - Consider existing if URL and HTTP method match
3. **Prefer existing title** - Keep user-defined titles over generated ones
4. **Mark as auto-generated** - Add `[Auto]` prefix to Memo field for tracking

---

## Step 4: Generate Test Case Status Documents

### 4.1 Output Locations

**File Naming Convention:**
- Convert target name to uppercase prefix
- Replace hyphens `-` with underscores `_`

| Target | Prefix | Output File |
|--------|--------|-------------|
| backend | BACKEND | `.claude-project/status/backend/BACKEND_TESTCASE_STATUS.md` |
| frontend | FRONTEND | `.claude-project/status/frontend/FRONTEND_TESTCASE_STATUS.md` |
| frontend-dashboard | FRONTEND_DASHBOARD | `.claude-project/status/frontend-dashboard/FRONTEND_DASHBOARD_TESTCASE_STATUS.md` |

Create directories if they don't exist.

### 4.2 Required Fields Per Test Case

Each test case row MUST include:

| Field | Description | Auto-Detection |
|-------|-------------|----------------|
| **Title** | Test name | From `it()`/`test()` |
| **URL** | Route or endpoint | From describe block or page object |
| **Status** | Test status | File/test existence |
| **Test Code** | File path with line | `path/file.spec.ts:42` |
| **Last Tested** | Date | User-maintained (preserve) |
| **Memo** | Notes | User-maintained (preserve) |
| **User Type** | Auth requirement | Path/URL pattern |

### 4.3 Document Template

```markdown
# Test Case Status: {App Name}

> **Application:** {backend/frontend/frontend-dashboard}
> **Test Framework:** {Jest/Playwright}
> **Last Generated:** {YYYY-MM-DD}
> **Total Test Cases:** {count}

---

## {Category Name}

| Title | URL | Status | Test Code | Last Tested | Memo | User Type |
|-------|-----|--------|-----------|-------------|------|-----------|
| {title} | {url} | {status} | `{file:line}` | | | {type} |

---

## Summary

| Category | Total | Complete | In Progress | Not Started | Skipped | Blocked |
|----------|-------|----------|-------------|-------------|---------|---------|
| {category} | X | X | X | X | X | X |
| **Total** | X | X | X | X | X | X |

---

## Test Infrastructure

### File Locations

| Type | Location |
|------|----------|
| E2E Tests | `{path}/test/**/*.spec.ts` |
| Page Objects | `{path}/test/pages/**/*.page.ts` |

### Running Tests

\`\`\`bash
cd {directory}

# Run all tests
npm run test:e2e

# Run specific file
npx {test-runner} {file}
\`\`\`
```

---

## Step 5: Status Detection Rules

| Condition | Status |
|-----------|--------|
| `it.skip()` or `test.skip()` found | Skipped |
| Test exists in spec file | Complete |
| Spec file exists but no matching test | In Progress |
| No spec file for this feature | Not Started |
| User-set in memo | Blocked |

---

## Step 6: Merge with Existing Data

When `{TARGET_PREFIX}_TESTCASE_STATUS.md` already exists:

1. **Read existing file** and parse table rows
2. **Preserve user data**:
   - `Last Tested` column values
   - `Memo` column values
   - Manual status overrides (if Memo contains "[Manual]")
3. **Update from test files**:
   - Refresh `Test Code` paths
   - Update `Status` based on current file scan
   - Add newly discovered tests
4. **Handle removed tests**:
   - Keep row with `[Removed]` prefix in Memo

**Match criteria**: Title + URL combination

---

## Step 7: Report Results

After processing all targets, output:

```markdown
## Test Case Generation Complete

### Processed Targets

| Target | Test Cases | Categories | Status |
|--------|------------|------------|--------|
| backend | XX | X | Generated |
| frontend | XX | X | Generated |
| frontend-dashboard | XX | X | Generated |

### Output Files

- `.claude-project/status/backend/BACKEND_TESTCASE_STATUS.md`
- `.claude-project/status/frontend/FRONTEND_TESTCASE_STATUS.md`
- `.claude-project/status/frontend-dashboard/FRONTEND_DASHBOARD_TESTCASE_STATUS.md`

### Statistics

| Metric | Backend | Frontend | Dashboard | Total |
|--------|---------|----------|-----------|-------|
| Complete | X | X | X | X |
| In Progress | X | X | X | X |
| Not Started | X | X | X | X |
| Skipped | X | X | X | X |
| Blocked | X | X | X | X |
| **Total** | X | X | X | X |

### Next Steps

1. Review generated test cases for accuracy
2. Update `Last Tested` dates after running tests
3. Add `Memo` notes for blocked/skipped tests
4. Implement tests marked as `Not Started`
```

---

## Backend Test Categories

Based on current backend test files:

| Category | Spec File | API Endpoints |
|----------|-----------|---------------|
| Authentication | `auth.e2e-spec.ts` | `/auth/*` |
| Users | `user.e2e-spec.ts` | `/users/*` |
| Admin | `admin.e2e-spec.ts` | `/admin/*` |

---

## Frontend Test Categories

Based on current frontend test files:

| Category | Spec Files | Routes |
|----------|------------|--------|
| Auth - Login | `login.spec.ts` | `/login` |
| Auth - Coach Signup | `coach-signup.spec.ts` | `/signup/coach` |
| Auth - Patient Signup | `patient-signup.spec.ts` | `/signup/patient` |
| Auth - Route Guards | `route-guards.spec.ts` | Multiple |
| Coach - Calendar | `calendar.spec.ts` | `/coach/calendar` |
| Coach - Chat | `chat.spec.ts` | `/coach/chat/*` |
| Coach - Patients | `patients.spec.ts` | `/coach/patients/*` |
| Patient - Home | `home.spec.ts` | `/patient/home` |
| Patient - Exercise | `exercise.spec.ts` | `/patient/exercise/*` |
| Patient - Chat | `chat.spec.ts` | `/patient/chat/*` |
| Patient - Survey | `survey.spec.ts` | `/patient/survey` |
| Patient - Notifications | `notifications.spec.ts` | `/patient/notifications` |

---

## Frontend-Dashboard Test Categories

Based on current frontend-dashboard test files:

| Category | Spec Files | Routes |
|----------|------------|--------|
| Auth | `login.spec.ts` | `/login` |
| Dashboard | `dashboard.spec.ts` | `/` |
| Users | `users.spec.ts` | `/users/*` |
| Exercises | `exercises.spec.ts` | `/exercises/*` |
| Surveys | `surveys.spec.ts` | `/surveys/*` |

---

## Error Handling

| Error | Action |
|-------|--------|
| Target directory not found | Skip with warning |
| No test directory | Generate with API-derived cases only |
| Parse error in spec file | Log warning, continue with partial data |
| Missing status directory | Create directory |

---

## Usage Examples

### Generate all test case documents

```
/generate-testcase
```

### Generate backend only

```
/generate-testcase backend
```

### Generate frontend-dashboard only

```
/generate-testcase frontend-dashboard
```

### Regenerate all (merges with existing)

```
/generate-testcase all
```

---

## Notes

- Backend tests use Jest patterns (`describe`/`it`)
- Frontend tests use Playwright patterns (`test.describe`/`test`)
- Line numbers point to individual test definitions
- Korean test names and memos are fully supported
- Swagger fallback uses PROJECT_API.md when backend is not running
- Always scan dynamically - avoid hardcoding test file lists
