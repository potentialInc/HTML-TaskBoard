# API Implementation Status: $PROJECT_NAME

## Overview

This document tracks the implementation status of all backend API endpoints.

---

## Authentication APIs

| Endpoint | Method | Status | Notes |
|----------|--------|--------|-------|
| `/auth/register` | POST | Pending | User registration |
| `/auth/login` | POST | Pending | JWT login |
| `/auth/logout` | POST | Pending | Invalidate token |
| `/auth/refresh` | POST | Pending | Refresh JWT |
| `/auth/forgot-password` | POST | Pending | Password reset request |
| `/auth/reset-password` | POST | Pending | Password reset confirm |

---

## User APIs

| Endpoint | Method | Status | Notes |
|----------|--------|--------|-------|
| `/users` | GET | Pending | List users (admin) |
| `/users/:id` | GET | Pending | Get user by ID |
| `/users/:id` | PATCH | Pending | Update user |
| `/users/:id` | DELETE | Pending | Delete user |
| `/users/me` | GET | Pending | Get current user |

---

## [Resource] APIs

<!-- Copy this section for each resource/entity -->

| Endpoint | Method | Status | Notes |
|----------|--------|--------|-------|
| `/[resource]` | GET | Pending | List all |
| `/[resource]` | POST | Pending | Create new |
| `/[resource]/:id` | GET | Pending | Get by ID |
| `/[resource]/:id` | PATCH | Pending | Update |
| `/[resource]/:id` | DELETE | Pending | Delete |

---

## Status Legend

| Status | Meaning |
|--------|---------|
| Pending | Not started |
| In Progress | Currently being implemented |
| Review | Implemented, needs review |
| Complete | Implemented and tested |
| Blocked | Waiting on dependency |

---

## Implementation Checklist

### Per Endpoint
- [ ] Controller method
- [ ] Service method
- [ ] Repository method (if needed)
- [ ] DTO validation
- [ ] Swagger documentation
- [ ] Unit tests
- [ ] E2E tests

### Global
- [ ] Error handling middleware
- [ ] Authentication guards
- [ ] Rate limiting
- [ ] Request logging
- [ ] API versioning

---

## Notes

<!-- Add implementation notes, blockers, decisions -->

