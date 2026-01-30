---
name: test-discover-config
description: Collect test configuration (ask user for credentials, auto-read the rest)
---

# Test Configuration Collection Skill

Collects project-specific test configuration:
- **Ask user**: Credentials only
- **Auto-read**: Login field, JWT secret, Token path

---

## When to Use

Run this skill before generating test code when:
- First time generating tests for a project
- Test fixtures don't exist yet
- Tests are failing due to credential/config mismatch

---

## Step 1: Ask User for Credentials

Use **AskUserQuestion**:

```
Question: "Please provide test credentials (e.g., admin/12341234)"
Header: "Credentials"
```

Collect for each role needed:
- ADMIN credentials (ID/password)
- COACH credentials (if applicable)
- PATIENT/USER credentials (if applicable)

---

## Step 2: Auto-read Login Field

Read LoginDto:

```
backend/src/modules/auth/dtos/login.dto.ts
```

Check if DTO has `username` or `email` field.

---

## Step 3: Auto-read JWT Secret

Read from `.env` file:

```
backend/.env
```

Search patterns:
- `AUTH_JWT_SECRET=`
- `JWT_SECRET=`
- `ACCESS_TOKEN_SECRET=`

Extract the value after `=`.

---

## Step 4: Auto-read Token Path

Read auth service:

```
backend/src/modules/auth/auth.service.ts
```

Find the login method return statement:

```typescript
// Pattern 1: Nested in data
return { data: { token: ... } }  // → tokenPath = 'data.token'

// Pattern 2: Flat
return { access_token: ... }  // → tokenPath = 'access_token'

// Pattern 3: data.access_token
return { data: { access_token: ... } }  // → tokenPath = 'data.access_token'
```

---

## Step 5: Output Configuration

```markdown
## Collected Test Configuration

### User Input
| Item | Value |
|------|-------|
| Admin credentials | {id}/{password} |
| Coach credentials | {id}/{password} |

### Auto-detected
| Item | Value | Source |
|------|-------|--------|
| Login field | {loginField} | LoginDto |
| JWT Secret | {jwtSecret} | .env |
| Token path | {tokenPath} | auth.service.ts |
```

---

## Output Variables

After collection:
- `credentials`: Array of {role, id, password}
- `loginField`: 'username' | 'email'
- `jwtSecret`: string (from .env)
- `tokenPath`: string (from auth service)

---

## Usage

Called by:
- `/generate-testcase` command (Step 2.5)
- `test/generate.md` skill (Step 0)
