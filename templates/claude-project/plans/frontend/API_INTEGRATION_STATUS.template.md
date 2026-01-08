# API Integration Status: $PROJECT_NAME

## Overview

This document tracks which frontend screens have been integrated with their required API endpoints.

---

## Integration Matrix

### Authentication

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Login | `POST /auth/login` | Pending | `authService.login()` |
| Register | `POST /auth/register` | Pending | `authService.register()` |
| Forgot Password | `POST /auth/forgot-password` | Pending | `authService.forgotPassword()` |
| Reset Password | `POST /auth/reset-password` | Pending | `authService.resetPassword()` |

### User Management

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Profile | `GET /users/me` | Pending | `userService.getCurrentUser()` |
| Profile Edit | `PATCH /users/:id` | Pending | `userService.updateUser()` |
| User List | `GET /users` | Pending | `userService.getUsers()` |

### [Feature]

<!-- Copy for each feature -->

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|

---

## Status Legend

| Status | Meaning |
|--------|---------|
| Pending | Not integrated |
| In Progress | Currently integrating |
| Complete | Integrated and working |
| Blocked | API not ready |

---

## API Services

### Service Files

| Service | Location | Endpoints |
|---------|----------|-----------|
| authService | `src/services/auth.ts` | login, register, logout, refresh |
| userService | `src/services/user.ts` | getUsers, getUser, updateUser, deleteUser |

### Service Implementation Checklist

Per service:
- [ ] Axios instance configured
- [ ] Request interceptors (auth token)
- [ ] Response interceptors (error handling)
- [ ] TypeScript types for requests/responses
- [ ] Error handling
- [ ] Loading state management

---

## Global Integration Setup

- [ ] Axios base configuration
- [ ] Auth token storage (cookies/localStorage)
- [ ] Token refresh mechanism
- [ ] Global error handler
- [ ] Request/Response logging (dev)
- [ ] API mocking for development

---

## Error Handling

### API Error Responses

| Error Code | Handling | User Message |
|------------|----------|--------------|
| 400 | Show validation errors | "Please check your input" |
| 401 | Redirect to login | "Session expired" |
| 403 | Show forbidden message | "Access denied" |
| 404 | Show not found | "Resource not found" |
| 500 | Show generic error | "Something went wrong" |

---

## Testing

### Integration Test Coverage

| Flow | Test Status | Notes |
|------|-------------|-------|
| Login → Dashboard | Pending | |
| Register → Verify → Login | Pending | |
| CRUD Operations | Pending | |

---

## Notes

<!-- Integration notes, API quirks, workarounds -->

