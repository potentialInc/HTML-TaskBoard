# Dashboard API Integration Status: $PROJECT_NAME

## Overview

This document tracks which dashboard screens have been integrated with their required API endpoints.

---

## Integration Matrix

### Authentication

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Admin Login | `POST /auth/admin/login` | Pending | `authService.adminLogin()` |
| 2FA Verify | `POST /auth/verify-2fa` | Pending | `authService.verify2FA()` |
| Logout | `POST /auth/logout` | Pending | `authService.logout()` |

### Dashboard Overview

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Stats Cards | `GET /admin/stats` | Pending | `statsService.getOverview()` |
| Recent Activity | `GET /admin/activity` | Pending | `activityService.getRecent()` |
| Charts Data | `GET /admin/analytics` | Pending | `analyticsService.getData()` |

### User Management

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| User List | `GET /admin/users` | Pending | `userService.getUsers()` |
| User Detail | `GET /admin/users/:id` | Pending | `userService.getUser()` |
| Create User | `POST /admin/users` | Pending | `userService.createUser()` |
| Update User | `PATCH /admin/users/:id` | Pending | `userService.updateUser()` |
| Delete User | `DELETE /admin/users/:id` | Pending | `userService.deleteUser()` |
| Bulk Actions | `POST /admin/users/bulk` | Pending | `userService.bulkAction()` |

### Roles & Permissions

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| List Roles | `GET /admin/roles` | Pending | `roleService.getRoles()` |
| Create Role | `POST /admin/roles` | Pending | `roleService.createRole()` |
| Update Role | `PATCH /admin/roles/:id` | Pending | `roleService.updateRole()` |
| Get Permissions | `GET /admin/permissions` | Pending | `roleService.getPermissions()` |

### Content Management

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Content List | `GET /admin/content` | Pending | `contentService.getList()` |
| Content Detail | `GET /admin/content/:id` | Pending | `contentService.getById()` |
| Create Content | `POST /admin/content` | Pending | `contentService.create()` |
| Update Content | `PATCH /admin/content/:id` | Pending | `contentService.update()` |
| Delete Content | `DELETE /admin/content/:id` | Pending | `contentService.delete()` |
| Publish Content | `POST /admin/content/:id/publish` | Pending | `contentService.publish()` |

### Media Library

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| List Media | `GET /admin/media` | Pending | `mediaService.getList()` |
| Upload Media | `POST /admin/media/upload` | Pending | `mediaService.upload()` |
| Delete Media | `DELETE /admin/media/:id` | Pending | `mediaService.delete()` |

### System Settings

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Get Settings | `GET /admin/settings` | Pending | `settingsService.get()` |
| Update Settings | `PATCH /admin/settings` | Pending | `settingsService.update()` |
| Audit Logs | `GET /admin/audit-logs` | Pending | `auditService.getLogs()` |

### Reports

| Screen | API Endpoint | Status | Service Method |
|--------|--------------|--------|----------------|
| Generate Report | `POST /admin/reports/generate` | Pending | `reportService.generate()` |
| Export Data | `POST /admin/reports/export` | Pending | `reportService.export()` |
| Get Templates | `GET /admin/reports/templates` | Pending | `reportService.getTemplates()` |

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
| authService | `src/services/auth.ts` | adminLogin, verify2FA, logout |
| userService | `src/services/admin/users.ts` | CRUD + bulk operations |
| roleService | `src/services/admin/roles.ts` | roles and permissions |
| contentService | `src/services/admin/content.ts` | content CRUD + publish |
| mediaService | `src/services/admin/media.ts` | upload, list, delete |
| settingsService | `src/services/admin/settings.ts` | get/update settings |
| auditService | `src/services/admin/audit.ts` | audit logs |
| reportService | `src/services/admin/reports.ts` | generate, export |
| statsService | `src/services/admin/stats.ts` | dashboard stats |
| analyticsService | `src/services/admin/analytics.ts` | charts data |

---

## Admin-Specific Considerations

### Authentication
- [ ] Admin-specific login endpoint
- [ ] Role validation (admin/superadmin)
- [ ] 2FA enforcement for admin accounts
- [ ] Session timeout configuration
- [ ] IP allowlisting (optional)

### Authorization
- [ ] Permission-based route guards
- [ ] UI element visibility by permission
- [ ] API error handling for 403

### Audit Trail
- [ ] Log all admin actions
- [ ] Track who made changes
- [ ] Timestamp all modifications

---

## Pagination & Filtering

### Standard Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | number | Page number (1-indexed) |
| `limit` | number | Items per page |
| `sort` | string | Sort field |
| `order` | asc/desc | Sort direction |
| `search` | string | Search query |
| `filters` | object | Field-specific filters |

---

## Error Handling

| Error Code | Handling | User Message |
|------------|----------|--------------|
| 400 | Show validation errors | "Please check your input" |
| 401 | Redirect to admin login | "Session expired" |
| 403 | Show permission denied | "You don't have access" |
| 404 | Show not found | "Resource not found" |
| 429 | Show rate limit | "Too many requests" |
| 500 | Show generic error | "Something went wrong" |

---

## Notes

<!-- Integration notes, API quirks, workarounds -->

