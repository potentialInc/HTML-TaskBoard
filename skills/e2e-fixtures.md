# E2E Test Fixtures Guide

Test data and fixture management for Playwright E2E tests.

## Table of Contents

- [Real API Testing Philosophy](#real-api-testing-philosophy)
- [Authentication Fixtures](#authentication-fixtures)
- [Test Data Management](#test-data-management)
- [Test Data Generators](#test-data-generators)
- [Best Practices](#best-practices)

---

## Real API Testing Philosophy

E2E tests run against the **real backend** - no mocking allowed.

### Why No Mocking?

1. **Catch Real Bugs**: Mocks hide integration issues that only appear with real APIs
2. **Test Real Behavior**: Mock responses may not match actual API behavior
3. **Maintain Test Value**: E2E tests verify the full user flow, not isolated components
4. **Reduce Maintenance**: No need to update mocks when APIs change

### Prerequisites

Before running tests:
1. Backend server must be running (`npm run start:dev` in backend/)
2. Database must have seeded test users (patient, coach, admin)
3. Frontend dev server must be running (`npm run dev` in frontend/)

### Error Testing Without Mocks

To test error scenarios, use real error conditions:
- Invalid credentials for auth errors (e.g., wrong password)
- Non-existent IDs for 404 errors (e.g., `/patient/exercise/99999`)
- Invalid data formats for validation errors (e.g., malformed email)
- Expired tokens for 401 errors (clear localStorage and attempt protected action)

---

## Authentication Fixtures

### Test Users

Tests use seeded data from the backend database:

```typescript
// frontend/test/fixtures/auth.fixture.ts

export const testUsers = {
  admin: {
    username: 'admin',
    email: 'admin@potentialai.com',
    password: '12341234',
    role: 'ADMIN',
    name: 'Admin User',
  },
  coach: {
    username: 'coach',
    email: 'coach@potentialai.com',
    password: '12341234',
    role: 'COACH',
    name: 'Coach User',
  },
  patient: {
    username: 'patient',
    email: 'patient@potentialai.com',
    password: '12341234',
    role: 'PATIENT',
    name: 'Patient User',
  },
};
```

### Authentication Helpers

```typescript
// Authenticate via UI - uses real backend API
export async function authenticateAsCoach(page: Page): Promise<void> {
  await page.goto('/');
  await page.getByPlaceholder('아이디를 입력하세요').fill(testUsers.coach.username);
  await page.getByPlaceholder('비밀번호를 입력하세요').fill(testUsers.coach.password);
  await page.getByRole('button', { name: '로그인' }).click();
  await page.waitForURL('/coach');
}

export async function authenticateAsPatient(page: Page): Promise<void> {
  await authenticate(page, testUsers.patient);
}

export async function authenticateAsAdmin(page: Page): Promise<void> {
  await authenticate(page, testUsers.admin);
}
```

### Usage in Tests

```typescript
import { test } from '@playwright/test';
import { authenticateAsCoach, clearAuthState } from '../../fixtures/auth.fixture';

test.describe('Coach Dashboard', () => {
  test.beforeEach(async ({ page }) => {
    await clearAuthState(page);
    await authenticateAsCoach(page);
  });

  test('should show dashboard', async ({ page }) => {
    await page.goto('/coach');
    // Test authenticated page with real data
  });
});
```

---

## Test Data Management

### Using Seeded Test Data

The backend database contains pre-seeded test data that tests can rely on:

| Role    | Username | Password   | Email                    |
|---------|----------|------------|--------------------------|
| Patient | patient  | 12341234   | patient@potentialai.com  |
| Coach   | coach    | 12341234   | coach@potentialai.com    |
| Admin   | admin    | 12341234   | admin@potentialai.com    |

### Test Data for Specific Features

Tests should use real data from the backend:

```typescript
test.describe('Patients List', () => {
  test('should display patients from backend', async ({ page }) => {
    await authenticateAsCoach(page);
    await page.goto('/coach/patients');

    // Assert that the list loads from real API
    await expect(page.locator('[data-testid="patient-list"]')).toBeVisible();

    // The actual patient names come from real backend data
    const patientCount = await page.locator('[data-testid="patient-item"]').count();
    expect(patientCount).toBeGreaterThan(0);
  });
});
```

### Creating Test Data

For tests that create new data (signup, form submissions), use unique identifiers:

```typescript
test('should register new patient', async ({ page }) => {
  const uniqueEmail = generateTestEmail(); // timestamp-based
  const uniquePhone = generateTestPhone();

  // Fill form with unique data
  await signupPage.fillEmail(uniqueEmail);
  await signupPage.fillPhone(uniquePhone);
  await signupPage.submit();

  // Real API creates the user
  await expect(page).toHaveURL(/\/patient/);
});
```

---

## Test Data Generators

### Unique Identifier Generators

```typescript
// frontend/test/fixtures/user.fixture.ts

// Generate unique test email (prevents conflicts)
export function generateTestEmail(): string {
  return `test-${Date.now()}@test.com`;
}

// Generate unique phone number
export function generateTestPhone(): string {
  const random = Math.floor(Math.random() * 90000000) + 10000000;
  return `010${random}`;
}

// Format phone to Korean standard
export function formatKoreanPhone(phone: string): string {
  return phone.replace(/(\d{3})(\d{4})(\d{4})/, '$1-$2-$3');
}
```

### Date Helpers

```typescript
// Get today's date in YYYY-MM-DD format
export function getTodayDate(): string {
  return new Date().toISOString().split('T')[0];
}

// Get date relative to today
export function getRelativeDate(daysOffset: number): string {
  const date = new Date();
  date.setDate(date.getDate() + daysOffset);
  return date.toISOString().split('T')[0];
}
```

---

## Best Practices

### 1. Clean State Before Tests

```typescript
test.beforeEach(async ({ page }) => {
  await clearAuthState(page);
  // Now authenticate fresh
});
```

### 2. Use Unique Data for Create Operations

```typescript
// Good - unique email per test
const uniqueEmail = generateTestEmail();

// Bad - hardcoded email that may conflict
const email = 'test@test.com';
```

### 3. Assert on Real Backend Responses

```typescript
test('should load patient exercises', async ({ page }) => {
  await authenticateAsPatient(page);
  await page.goto('/patient/exercise');

  // Wait for real API response
  await page.waitForLoadState('networkidle');

  // Assert on actual data structure
  await expect(page.locator('[data-testid="exercise-list"]')).toBeVisible();
});
```

### 4. Handle Loading States

```typescript
test('should show loading then data', async ({ page }) => {
  await authenticateAsPatient(page);
  await page.goto('/patient/exercise');

  // May see loading briefly
  // Then real data appears
  await expect(page.getByText(/운동/)).toBeVisible({ timeout: 10000 });
});
```

### 5. Test Error States with Real Conditions

```typescript
test('should show error for invalid patient ID', async ({ page }) => {
  await authenticateAsCoach(page);

  // Navigate to non-existent patient (real 404)
  await page.goto('/coach/patient/99999');

  // Assert error handling
  await expect(page.getByText(/찾을 수 없습니다|not found/i)).toBeVisible();
});

test('should show auth error for expired session', async ({ page }) => {
  await authenticateAsPatient(page);

  // Clear auth to simulate expired session
  await page.evaluate(() => localStorage.removeItem('token'));

  // Try to access protected resource
  await page.goto('/patient/exercise');

  // Should redirect to login
  await expect(page).toHaveURL('/');
});
```

---

## Related Files

- [e2e-test-generator.md](e2e-test-generator.md) - Test patterns
- [e2e-page-objects.md](e2e-page-objects.md) - Page object patterns
- [auth.fixture.ts](../../../frontend/test/fixtures/auth.fixture.ts) - Auth implementation
- [user.fixture.ts](../../../frontend/test/fixtures/user.fixture.ts) - Test data generators
