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

### 2026-01-28: Google OAuth Integration - Implicit Flow

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: When implementing Google OAuth with implicit flow (response_type=token):

1. **Frontend stores state in localStorage** (not sessionStorage) - sessionStorage can be cleared during redirects
2. **Google returns access_token in URL hash**, not id_token
3. **Backend validates via userinfo endpoint**: `https://www.googleapis.com/oauth2/v2/userinfo` with Bearer token
4. **Do NOT use tokeninfo endpoint for access tokens** - that's for id_tokens

**Code Pattern**:
```typescript
// Frontend - store OAuth state
localStorage.setItem('oauth_state', state);

// Backend - validate access token
const response = await axios.get('https://www.googleapis.com/oauth2/v2/userinfo', {
  headers: { Authorization: `Bearer ${token}` }
});
```

---

### 2026-01-28: Backend Social Login Enum Values

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: The backend SocialLoginTypeEnum uses **numeric values**, not strings:
- APPLE = 1
- KAKAO = 2
- GOOGLE = 3
- NAVER = 4

Frontend must send these numeric values in the `socialLoginType` field.

---

### 2026-01-28: Social Login New User Registration

**Type**: pattern
**Confidence**: HIGH
**Source**: Session reflection - design-flow project

**Description**: When registering new users via social login, the backend requires `termsAndConditionsAccepted: true` to create the account. Without this flag, the backend returns "Please signup first" error.

---
