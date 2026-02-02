---
name: test-generate
description: Generate test code for missing test cases
---

# Test Generate Skill

Creates test code for test cases that are documented but not yet implemented.

---

## Input

- `target`: The project target (frontend, backend, frontend-dashboard)
- `TESTCASE_STATUS.md` with "Not Started" entries (from `analyze.md` skill)

---

## Step 0: Collect Test Configuration (CRITICAL)

**Before generating any test code, collect project-specific configuration.**

### 0.1 Ask User for Credentials

Use **AskUserQuestion** to collect:

```
Question: "Please provide test credentials (e.g., admin/12341234)"
```

Required information:
- Admin: ID + password
- Other roles as needed (Coach, Patient, User)

### 0.2 Auto-read Login Field from LoginDto

Read LoginDto to determine login field:
```
backend/src/modules/auth/dtos/login.dto.ts
```

Check if DTO uses `username` or `email`.

### 0.3 Auto-read JWT Secret from .env

Read JWT secret from `.env` file:
```
backend/.env
```

Look for: `JWT_SECRET`, `AUTH_JWT_SECRET`, `ACCESS_TOKEN_SECRET`

### 0.4 Auto-read Token Path from Auth Service

Read auth service to find token path in response:
```
backend/src/modules/auth/auth.service.ts
```

Look for return statement structure (e.g., `data.token`, `access_token`)

### Rules
1. **Ask user** for credentials only
2. **Auto-read** login field from LoginDto
3. **Auto-read** JWT secret from .env
4. **Auto-read** token path from auth service

---

## Step 0.5: Read Actual API Information (Backend Only)

**CRITICAL: Read actual API behavior before generating backend tests.**

### Option 1: Swagger (Backend Running)

If backend is running at `http://localhost:4000`:

1. Check if Swagger is accessible:
   ```bash
   curl -s http://localhost:4000/api-json | head -1
   ```

2. If accessible, fetch and parse Swagger JSON:
   - Endpoint paths and HTTP methods
   - Request body schemas (required fields, types)
   - Response schemas (success/error formats)
   - Status codes (200, 201, 400, 401, 403, 404)
   - Authentication requirements

### Option 2: Controller Files (Backend Not Running)

Read controller files directly:
```
backend/src/modules/*/*.controller.ts
backend/src/modules/*/controllers/*.controller.ts
```

Extract from decorators:
| Decorator | Information |
|-----------|-------------|
| `@Get`, `@Post`, `@Patch`, `@Delete` | HTTP method |
| `@Controller('path')` | Base path |
| `@HttpCode(HttpStatus.XXX)` | Expected status code |
| `@ApiSwagger({ responseDto: XXX })` | Response DTO class |
| `@AdminOnly`, `@UseGuards(JwtAuthGuard)` | Auth requirement |

### Option 3: PROJECT_API.md (Fallback)

If neither Swagger nor controller files are available:
```
.claude-project/docs/PROJECT_API.md
```

### API Information to Collect

| Field | Source | Example |
|-------|--------|---------|
| Endpoints | Controller decorators | `GET /admin/dashboard` |
| Status codes | `@HttpCode` decorator | 200, 201, 204 |
| Response format | ResponsePayloadDto | `{ success, statusCode, message, data }` |
| Required auth | Guards/decorators | Admin only, Coach only |
| Request body | DTO classes | `{ username, password }` |

### Standard Response Format (NestJS)

Most endpoints return `ResponsePayloadDto`:
```typescript
{
  success: boolean;
  statusCode: number;
  message: string;
  data: T;
  timestamp: string;
}
```

Use this format when generating test assertions.

---

## Step 1: Read Test Status

Read the test status file:
```
.claude-project/status/{target}/{TARGET_PREFIX}_TESTCASE_STATUS.md
```

**File Naming Convention:**
- Convert target name to uppercase prefix
- Replace hyphens `-` with underscores `_`

| Target | Prefix | File |
|--------|--------|------|
| backend | BACKEND | `BACKEND_TESTCASE_STATUS.md` |
| frontend | FRONTEND | `FRONTEND_TESTCASE_STATUS.md` |
| frontend-dashboard | FRONTEND_DASHBOARD | `FRONTEND_DASHBOARD_TESTCASE_STATUS.md` |

Example: `.claude-project/status/backend/BACKEND_TESTCASE_STATUS.md`

### Extract Tests to Generate

Parse the markdown table and filter by status:

```
For each row in test table:
  If status == "Not Started":
    Add to tests_to_generate[]
```

Group by category/endpoint for efficient generation.

### If No STATUS File Exists

Run `analyze.md` skill first to:
1. Scan application code for features
2. Create initial TESTCASE_STATUS.md with "Not Started" entries

---

## Step 2: Read Existing Test Patterns

### For Backend (Jest/NestJS)

Read existing E2E test files to understand patterns:
```
backend/test/*.e2e-spec.ts
```

Extract:
- Import statements
- Test setup (beforeAll, beforeEach)
- Authentication patterns
- Request/response patterns
- Assertion styles

### For Frontend (Playwright)

Read existing test files:
```
{target}/test/tests/**/*.spec.ts
```

Read page objects:
```
{target}/test/pages/**/*.page.ts
```

Extract:
- Import statements
- Authentication fixtures
- Page object usage
- Locator patterns
- Assertion styles

---

## Step 3: Generate Test Code

### For Backend Tests

**Template:**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('{Category}', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();

    // Get auth token - use credentials discovered from user seed file
    // Login field ({loginField}) determined from LoginDto
    const loginResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ {loginField}: '{admin_credential}', password: '{admin_password}' });
    authToken = loginResponse.body.{tokenPath}; // Token path from auth service
  });

  afterAll(async () => {
    await app.close();
  });

  describe('{HTTP_METHOD} {endpoint}', () => {
    it('{test_title}', async () => {
      const response = await request(app.getHttpServer())
        .{method}('{endpoint}')
        .set('Authorization', `Bearer ${authToken}`)
        .send({payload});

      expect(response.status).toBe({expected_status});
      expect(response.body).toHaveProperty('{expected_property}');
    });
  });
});
```

### For Frontend Tests (Playwright)

**Timeout Settings:**
```typescript
// In playwright.config.ts - increase global timeout
export default defineConfig({
  timeout: 60000, // 60 seconds (default is 30s)
  expect: {
    timeout: 10000, // 10 seconds for assertions
  },
});

// In individual test file - for slow tests
test.setTimeout(60000); // 60 seconds for this file

// For specific slow test
test('slow loading test', async ({ page }) => {
  test.slow(); // Triples the timeout (180s)
  // ...
});
```

**Template:**
```typescript
import { test, expect } from '@playwright/test';
import { {PageObject} } from '../../pages/{category}/{page}.page';
import { authenticateAs{Role} } from '../../fixtures/auth.fixture';

// Increase timeout for this test file if needed
test.setTimeout(60000);

test.describe('{Category}', () => {
  let page: {PageObject};

  test.beforeEach(async ({ page: browserPage }) => {
    await authenticateAs{Role}(browserPage);
    page = new {PageObject}(browserPage);
    await page.navigate();
  });

  test('{test_title}', async ({ page: browserPage }) => {
    // Arrange
    await page.{setup_action}();

    // Act
    await page.{test_action}();

    // Assert
    await expect(page.{element}).toBeVisible({ timeout: 15000 }); // 15s for slow elements
    await expect(browserPage).toHaveURL(/{expected_url}/);
  });
});
```

---

## Step 4: Determine File Placement

### Rule 1: Add to existing file if category matches

```
If existing spec file covers same category:
  Add new test to existing file
Else:
  Create new spec file
```

### Rule 2: Follow naming conventions

| Target | Pattern |
|--------|---------|
| backend | `{category}.e2e-spec.ts` |
| frontend | `{category}.spec.ts` |
| frontend-dashboard | `{category}.spec.ts` |

### Rule 3: Organize by feature

```
test/tests/
├── auth/
│   ├── login.spec.ts
│   └── signup.spec.ts
├── {feature}/
│   └── {feature}.spec.ts
```

---

## Step 5: Generate Page Objects (if needed)

For frontend tests, check if page object exists:

```
{target}/test/pages/{category}/{page}.page.ts
```

If not exists, generate:

```typescript
import { Page, Locator } from '@playwright/test';
import { BasePage } from '../base.page';

export class {PageName}Page extends BasePage {
  readonly url = '{route}';

  // Locators
  readonly {element}Button: Locator;
  readonly {element}Input: Locator;

  constructor(page: Page) {
    super(page);
    this.{element}Button = page.locator('[data-testid="{element}"]');
    this.{element}Input = page.locator('input[name="{element}"]');
  }

  async navigate() {
    await this.page.goto(this.url);
    await this.page.waitForLoadState('networkidle');
  }

  async {action}() {
    await this.{element}Button.click();
  }
}
```

---

## Step 6: Write Files

For each generated test:

1. **If adding to existing file:**
   - Read existing file
   - Find appropriate location (inside describe block)
   - Insert new test
   - Write file

2. **If creating new file:**
   - Create directory if needed
   - Write new spec file
   - Write page object if needed

---

## Step 7: Verify Generated Code

After writing files:

1. **Check TypeScript compilation:**
   ```bash
   cd {target} && npx tsc --noEmit
   ```

2. **Check imports are valid:**
   - Verify page objects exist
   - Verify fixtures are available

3. **Run linter:**
   ```bash
   cd {target} && npm run lint
   ```

---

## Step 8: Report Results

```markdown
## Test Code Generation: {target}

**Generated at:** {timestamp}

---

### New Test Files Created

| File | Tests Added | Category |
|------|-------------|----------|
| `{file_path}` | {count} | {category} |

### Existing Files Updated

| File | Tests Added | Total Tests |
|------|-------------|-------------|
| `{file_path}` | {added} | {total} |

### Page Objects Created

| Page Object | File | URL |
|-------------|------|-----|
| {PageName}Page | `{file_path}` | {route} |

---

### Summary

| Metric | Count |
|--------|-------|
| Tests Generated | {total_tests} |
| New Files | {new_files} |
| Updated Files | {updated_files} |
| Page Objects | {page_objects} |

---

### Next Steps

1. Review generated test code
2. Add specific assertions as needed
3. Run tests with `/test {target} --run`
```

---

## Output

Returns:
- List of created/modified files
- Summary of generated tests
- Any errors or warnings

---

## Test Generation Rules

### Rule 1: Match existing style
- Use same assertion library
- Follow existing naming conventions
- Match indentation and formatting

### Rule 2: Use meaningful assertions
- Don't just check "page loads"
- Verify actual behavior
- Check error states

### Rule 3: Handle authentication
- Use appropriate fixtures
- Test both authenticated and public routes
- Handle different user roles

### Rule 4: Keep tests independent
- Each test should be self-contained
- Don't rely on other tests' state
- Clean up after tests if needed

---

## Error Handling

| Error | Action |
|-------|--------|
| No TESTCASE_STATUS.md found | Run analyze skill first to create it |
| No "Not Started" tests found | All tests implemented, nothing to generate |
| Page object missing | Generate basic page object |
| TypeScript error | Report error, suggest fix |
| Lint error | Auto-fix if possible, report otherwise |

---

## Test Setup Troubleshooting (Backend E2E)

**CRITICAL: Use this section when tests fail due to setup issues.**

### Common Error: Module Not Found (404 for all endpoints)

**Symptom:**
```
All tests return 404 Not Found
```

**Root Cause:** The test module doesn't import the feature module being tested.

**Fix:**
1. Locate test app factory:
   ```
   backend/test/setup/test-app.factory.ts
   ```
2. Add missing module to imports:
   ```typescript
   imports: [
     // ...existing modules
     {MissingModule},  // Add the module being tested
   ]
   ```

### Common Error: Can't Resolve Dependencies

**Symptom:**
```
Nest can't resolve dependencies of {Service/Interceptor} (?).
Please make sure that the argument {Dependency} at index [0] is available in the {Module} context.
```

**Root Cause:** A service or interceptor depends on another service that isn't provided.

**Fix Options:**

**Option 1: Import the module providing the dependency**
```typescript
imports: [
  // ...existing modules
  {DependencyModule},  // Add module that provides the dependency
]
```

**Option 2: Create a mock module (for external services like Redis, Mail)**
```typescript
// test/mocks/{service}.module.mock.ts
@Global()
@Module({
  providers: [
    {
      provide: {OriginalService},
      useClass: Mock{Service},
    },
  ],
  exports: [{OriginalService}],
})
export class Mock{Service}Module {}
```

Then add to test app factory:
```typescript
imports: [
  Mock{Service}Module,  // Add before modules that use it
  // ...other modules
]
```

### Common Error: User/Data Not Found

**Symptom:**
```
TypeError: Cannot read properties of undefined (reading 'id')
```

**Root Cause:** Test data seeding failed or doesn't match test expectations.

**Fix:**
1. Check seed function returns all expected users:
   ```typescript
   // test/fixtures/user.fixture.ts
   export async function seedTestUsers(dataSource: DataSource) {
     return { admin, coach, patient, user };  // All roles must be returned
   }
   ```
2. Verify beforeEach calls seedTestUsers properly:
   ```typescript
   beforeEach(async () => {
     await cleanDatabase(dataSource);
     users = await seedTestUsers(dataSource);  // Must await
   });
   ```

### Common Error: Status Code Mismatch

**Symptom:**
```
expected 200 "OK", got 201 "Created"
expected 404 "Not Found", got 400 "Bad Request"
```

**Root Cause:** Test assertions don't match actual API status codes.

**Fix:**
1. Check actual API status codes from controller:
   ```typescript
   @Post()
   @HttpCode(HttpStatus.CREATED)  // 201, not 200
   create() {}
   ```
2. Update test expectations to match:
   ```typescript
   .expect(201)  // Not .expect(200)
   ```

### Module Import Order Checklist

When adding modules to test setup, follow this order:

```typescript
imports: [
  // 1. Core configuration modules
  ConfigModule.forRoot({ ... }),
  TypeOrmModule.forRoot({ ... }),

  // 2. Infrastructure mock modules (Redis, Mail, etc.)
  MockRedisCacheModule,  // Before any module that uses cache

  // 3. Shared/global modules
  PassportModule.register({ ... }),

  // 4. Feature modules (order matters for dependencies)
  UserModule,
  AuthModule,
  NotificationModule,  // Before modules that use notifications
  ExercisesModule,
  // ...other modules

  // 5. Admin/aggregate modules last
  AdminModule,  // Usually depends on multiple modules
]
```

### Quick Diagnosis Commands

```bash
# Check which module is missing
cd backend && npm run test:e2e 2>&1 | grep "can't resolve"

# Check 404 errors
cd backend && npm run test:e2e 2>&1 | grep "404"

# Run single test file
cd backend && npx jest --config ./test/jest-e2e.json test/e2e/{name}.e2e-spec.ts
```
