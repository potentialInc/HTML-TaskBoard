---
description: Create a complete new project with Claude config and boilerplate code
argument-hint: "<project-name> [--docs-only]"
---

You are a full project setup assistant. This command creates a new project with the shared claude-workflow as `.claude` submodule.

## Step 0: Parse Arguments and Select Mode

### 0.1 Parse Project Name

Parse `$ARGUMENTS` for `$PROJECT_NAME`.

```
Examples:
  /new-project my-app           → Full setup
  /new-project my-app --docs-only → Documentation only
```

If no project name provided, ask:
```
What is the project name? (e.g., monkey, coaching-platform)
```

Store as `$PROJECT_NAME` (lowercase, hyphenated).

### 0.2 Determine Setup Mode

Check if `--docs-only` flag is present in `$ARGUMENTS`:

**If `--docs-only` flag is present:**
```bash
DOCS_ONLY_MODE=true
```

**If `--docs-only` flag is NOT present:**

Use **AskUserQuestion** to ask the user:

```
Question: "What type of setup do you want to perform?"
Header: "Setup Mode"
Options:
  1. Full Project Setup (Recommended)
     Description: "Create new project with boilerplate code, .claude config, and documentation"

  2. Documentation Only
     Description: "Set up documentation for existing project (requires .claude/ submodule)"
```

Store result:
- Option 1 selected → `$DOCS_ONLY_MODE = false`
- Option 2 selected → `$DOCS_ONLY_MODE = true`

### 0.3 Display Selected Mode

Display confirmation:
```
=== Setup Mode Selected ===
Mode: ${DOCS_ONLY_MODE ? "Documentation Only" : "Full Project Setup"}
Project: $PROJECT_NAME
```

## Step 0.5: Detect and Migrate Resources

**This step ALWAYS runs in both normal and --docs-only modes.**

This step automatically detects existing project resources (HTML prototypes, PRD documents) at the root directory and migrates them to the proper locations within `.claude-project/`.

### 0.5.1 Check for Existing Resources

```bash
# Check for HTML folder
HTML_FOLDER_EXISTS=false
HTML_FILE_COUNT=0
if [ -d "HTML" ]; then
  HTML_FOLDER_EXISTS=true
  HTML_FILE_COUNT=$(find HTML -name "*.html" 2>/dev/null | wc -l | tr -d ' ')
fi

# Check for PRD PDF
PRD_PDF_EXISTS=false
if [ -f "prd.pdf" ]; then
  PRD_PDF_EXISTS=true
fi
```

### 0.5.2 Report Detected Resources

If resources are found, display:

```
=== Detected Project Resources ===

HTML Folder:  ${HTML_FOLDER_EXISTS ? "Found ($HTML_FILE_COUNT HTML files)" : "Not found"}
PRD PDF:      ${PRD_PDF_EXISTS ? "Found (prd.pdf)" : "Not found"}
```

### 0.5.3 Execute Migration (Automatic)

Automatically migrate resources without prompting:

```bash
# Create target directories
mkdir -p .claude-project/resources
mkdir -p .claude-project/prd

# Move HTML folder
if [ "$HTML_FOLDER_EXISTS" = true ]; then
  mv HTML .claude-project/resources/HTML
  echo "Migrated: HTML/ → .claude-project/resources/HTML/ ($HTML_FILE_COUNT files)"
fi

# Move PRD PDF
if [ "$PRD_PDF_EXISTS" = true ]; then
  mv prd.pdf .claude-project/prd/prd.pdf
  echo "Migrated: prd.pdf → .claude-project/prd/prd.pdf"
fi
```

### 0.5.4 Auto-Detect Tech Stack from Resources

After migration, analyze resources to pre-populate tech stack variables.

#### From PRD PDF (if exists):

Read the PDF and search for "Project Type" section:

```
Detection Patterns:
  - "Backend - NestJS" or "Backend: NestJS" → $BACKEND_DETECTED = "nestjs"
  - "Backend - Django" or "Backend: Django" → $BACKEND_DETECTED = "django"
  - "Web Application - React" → add "react" to $FRONTENDS_DETECTED
  - "Mobile App - React Native" → add "react-native" to $FRONTENDS_DETECTED
  - "Admin Dashboard - React" → add "admin" to $DASHBOARDS_DETECTED
  - "Operations Dashboard - React" → add "operations" to $DASHBOARDS_DETECTED
  - "Analytics Dashboard - React" → add "analytics" to $DASHBOARDS_DETECTED
  - "Coach Dashboard - React" → add "coach" to $DASHBOARDS_DETECTED
```

#### From HTML Files (if exist):

Analyze first 3 HTML files for framework hints:

```bash
# Check for TailwindCSS CDN (strongly suggests React)
if grep -l "tailwindcss.com" .claude-project/resources/HTML/*.html 2>/dev/null | head -1; then
  FRAMEWORK_HINT="react"
  CSS_FRAMEWORK="tailwindcss"
fi

# Check for Vue.js
if grep -l "vue.js\|vue.min.js" .claude-project/resources/HTML/*.html 2>/dev/null | head -1; then
  FRAMEWORK_HINT="vue"
fi

# Check for Angular
if grep -l "angular" .claude-project/resources/HTML/*.html 2>/dev/null | head -1; then
  FRAMEWORK_HINT="angular"
fi
```

### 0.5.5 Report Detection Results

```
=== Tech Stack Auto-Detection Results ===

From PRD (prd.pdf):
  Backend:    $BACKEND_DETECTED (or "Not detected")
  Frontend:   $FRONTENDS_DETECTED (or "Not detected")
  Dashboards: $DASHBOARDS_DETECTED (or "None detected")

From HTML Analysis:
  Framework Hint: $FRAMEWORK_HINT (or "None")
  CSS Framework:  $CSS_FRAMEWORK (or "None")
  File Count:     $HTML_FILE_COUNT files

These values will be used as defaults in Step 1.
```

### 0.5.6 Populate Documentation from PRD

After stack detection, automatically populate `.claude-project/docs/` with detailed information extracted from the PRD.

#### 0.5.6.0 Organize HTML Files by Type

Before populating docs, organize HTML files into subfolders by user type:

```bash
cd .claude-project/resources/HTML

# Create type folders
mkdir -p auth business-user data-analyst ops-manager admin settings modals

# Move by filename pattern (simple numeric prefix mapping)
mv 01-*.html 02-*.html 03-*.html 04-*.html 05-*.html 06-*.html auth/ 2>/dev/null
mv 07-*.html 08-*.html 09-*.html 10-*.html 11-*.html 12-*.html business-user/ 2>/dev/null
mv 13-*.html 14-*.html 15-*.html 16-*.html 17-*.html data-analyst/ 2>/dev/null
mv 18-*.html 19-*.html 20-*.html 21-*.html 22-*.html ops-manager/ 2>/dev/null
mv 23-*.html 24-*.html 25-*.html 26-*.html 27-*.html 28-*.html 29-*.html 30-*.html admin/ 2>/dev/null
mv settings-*.html settings/ 2>/dev/null
mv modal-*.html modals/ 2>/dev/null
# Remaining files stay at root (misc screens like alert-details, user-details, etc.)

cd -
```

**Result structure:**
```
.claude-project/resources/HTML/
├── auth/           # 01-06 (landing, login, signup, forgot-pwd, reset-pwd, email-verify)
├── business-user/  # 07-12 (home, dashboard-list, dashboard-view, reports, alerts, settings)
├── data-analyst/   # 13-17 (home, builder, query, models, model-editor)
├── ops-manager/    # 18-22 (dashboard, team, member-detail, workflows, sla)
├── admin/          # 23-30 (dashboard, users, sources, keys, branding, health, audit, billing)
├── settings/       # settings-*.html
├── modals/         # modal-*.html
└── *.html          # remaining detail/misc screens
```

#### 0.5.6.1 Prepare PRD Content for PROJECT_KNOWLEDGE.md

**Important**: The PROJECT_KNOWLEDGE.template.md already contains comprehensive Architecture sections for all frameworks (NestJS, Django, React, React Native) with detailed directory structures, request lifecycles, and patterns. These will be automatically populated in Step 7 when templates are copied and placeholders are replaced.

**This step only extracts PRD-specific content** to populate the following sections:

Read PRD and extract the following content:

```
Extract from PRD Part 1:
  - Description → Overview section
  - Goals → Goals list (numbered)
  - User Types → User Types table with permissions
  - Terminology → Terminology table (Term | Definition)
  - 3rd Party API List → External Services table
  - System Modules → Key Features summary
```

**Store extracted content in memory for Step 7.8** (will be added to PROJECT_KNOWLEDGE.md after template is copied):

```markdown
## Overview Section Content:
[Extract project description from PRD Part 1]

## Goals Section Content:
1. [Goal 1 from PRD]
2. [Goal 2 from PRD]
3. [Goal 3 from PRD]
4. [Goal 4 from PRD]

## User Types Section Content:
| Role | Permissions |
|------|-------------|
| Business User | [Extract from PRD User Types] |
| Data Analyst | [Extract from PRD User Types] |
| Operations Manager | [Extract from PRD User Types] |
| System Admin | [Extract from PRD User Types] |

## Terminology Section Content:
| Term | Definition |
|------|------------|
| Widget | [From PRD Terminology] |
| Data Model | [From PRD Terminology] |
| KPI | [From PRD Terminology] |
| Pipeline | [From PRD Terminology] |
| Threshold | [From PRD Terminology] |
| Connector | [From PRD Terminology] |
| SLA | [From PRD Terminology] |
| Drill-down | [From PRD Terminology] |
| Time Series | [From PRD Terminology] |
| Aggregation | [From PRD Terminology] |

## External Services Section Content:
| Service | Purpose | Documentation |
|---------|---------|---------------|
| [Service from PRD] | [Purpose from PRD] | [Link from PRD] |
```

**Note**: The Tech Stack section will be automatically populated from `{BACKEND}` and `{FRONTENDS}` placeholders. The Architecture section is already comprehensive in the template and requires no modification.

#### 0.5.6.2 Update PROJECT_API.md

Extract API structure from PRD System Modules and Page Architecture:

```markdown
## Endpoints

### Auth

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register` | Register new user | No |
| POST | `/auth/login` | Login user | No |
| POST | `/auth/forgot-password` | Send password reset email | No |
| POST | `/auth/reset-password` | Reset password with token | No |
| POST | `/auth/refresh` | Refresh access token | Yes |
| POST | `/auth/logout` | Logout user | Yes |
| GET | `/auth/google` | Google OAuth redirect | No |
| GET | `/auth/okta` | Okta SSO redirect | No |

### Dashboards

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/dashboards` | List user's dashboards | Yes |
| POST | `/dashboards` | Create new dashboard | Yes (Analyst) |
| GET | `/dashboards/:id` | Get dashboard by ID | Yes |
| PATCH | `/dashboards/:id` | Update dashboard | Yes (Owner) |
| DELETE | `/dashboards/:id` | Delete dashboard | Yes (Owner) |
| POST | `/dashboards/:id/duplicate` | Duplicate dashboard | Yes |
| POST | `/dashboards/:id/share` | Share dashboard with users | Yes (Owner) |
| GET | `/dashboards/:id/widgets` | Get dashboard widgets | Yes |
| POST | `/dashboards/:id/widgets` | Add widget to dashboard | Yes (Analyst) |
| PATCH | `/dashboards/:id/widgets/:widgetId` | Update widget | Yes (Analyst) |
| DELETE | `/dashboards/:id/widgets/:widgetId` | Remove widget | Yes (Analyst) |

### Alerts

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/alerts` | List user's alerts | Yes |
| POST | `/alerts` | Create new alert | Yes |
| GET | `/alerts/:id` | Get alert details | Yes |
| PATCH | `/alerts/:id` | Update alert config | Yes |
| DELETE | `/alerts/:id` | Delete alert | Yes |
| POST | `/alerts/:id/acknowledge` | Acknowledge triggered alert | Yes |
| POST | `/alerts/:id/snooze` | Snooze alert | Yes |
| POST | `/alerts/:id/resolve` | Resolve alert | Yes |

### Data Sources

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/data-sources` | List connected data sources | Yes (Admin) |
| POST | `/data-sources` | Add new data source | Yes (Admin) |
| GET | `/data-sources/:id` | Get data source details | Yes (Admin) |
| PATCH | `/data-sources/:id` | Update data source | Yes (Admin) |
| DELETE | `/data-sources/:id` | Remove data source | Yes (Admin) |
| POST | `/data-sources/:id/test` | Test connection | Yes (Admin) |
| POST | `/data-sources/:id/sync` | Sync metadata | Yes (Admin) |
| GET | `/data-sources/:id/tables` | List available tables | Yes |

### Reports

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/reports` | List scheduled reports | Yes |
| POST | `/reports` | Create scheduled report | Yes |
| GET | `/reports/:id` | Get report details | Yes |
| PATCH | `/reports/:id` | Update report schedule | Yes |
| DELETE | `/reports/:id` | Delete scheduled report | Yes |
| GET | `/reports/:id/history` | Get delivery history | Yes |
| POST | `/reports/:id/regenerate` | Regenerate report | Yes |

### Query Editor (Analyst)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/queries/execute` | Execute SQL query | Yes (Analyst) |
| GET | `/queries/history` | Get query history | Yes (Analyst) |
| POST | `/queries/save` | Save query | Yes (Analyst) |
| GET | `/queries/saved` | List saved queries | Yes (Analyst) |

### Data Models (Analyst)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/models` | List data models | Yes |
| POST | `/models` | Create data model | Yes (Analyst) |
| GET | `/models/:id` | Get model details | Yes |
| PATCH | `/models/:id` | Update model | Yes (Analyst) |
| DELETE | `/models/:id` | Delete model | Yes (Analyst) |
| POST | `/models/:id/validate` | Validate model | Yes (Analyst) |

### Operations (Ops Manager)

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/ops/dashboard` | Get operations metrics | Yes (Ops) |
| GET | `/ops/team` | List team members | Yes (Ops) |
| GET | `/ops/team/:id` | Get team member details | Yes (Ops) |
| GET | `/ops/workflows` | List pending workflows | Yes (Ops) |
| POST | `/ops/workflows/:id/approve` | Approve workflow | Yes (Ops) |
| POST | `/ops/workflows/:id/reject` | Reject workflow | Yes (Ops) |
| GET | `/ops/sla` | Get SLA status | Yes (Ops) |

### Admin

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/admin/stats` | Get admin statistics | Yes (Admin) |
| GET | `/admin/users` | List all users | Yes (Admin) |
| POST | `/admin/users` | Create user | Yes (Admin) |
| GET | `/admin/users/:id` | Get user details | Yes (Admin) |
| PATCH | `/admin/users/:id` | Update user | Yes (Admin) |
| DELETE | `/admin/users/:id` | Delete user | Yes (Admin) |
| POST | `/admin/users/:id/reset-password` | Reset user password | Yes (Admin) |
| GET | `/admin/api-keys` | List API keys | Yes (Admin) |
| POST | `/admin/api-keys` | Generate API key | Yes (Admin) |
| DELETE | `/admin/api-keys/:id` | Revoke API key | Yes (Admin) |
| GET | `/admin/audit-log` | Get audit log | Yes (Admin) |
| GET | `/admin/health` | Get system health | Yes (Admin) |
| GET | `/admin/branding` | Get branding settings | Yes (Admin) |
| PATCH | `/admin/branding` | Update branding | Yes (Admin) |

### Billing

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/billing/subscription` | Get current subscription | Yes (Admin) |
| POST | `/billing/subscribe` | Subscribe to plan | Yes (Admin) |
| PATCH | `/billing/subscription` | Update subscription | Yes (Admin) |
| GET | `/billing/invoices` | List invoices | Yes (Admin) |
| GET | `/billing/invoices/:id` | Get invoice details | Yes (Admin) |
```

#### 0.5.6.3 Prepare PRD-Specific Tables for PROJECT_DATABASE.md

**Important**: The PROJECT_DATABASE.template.md already contains:
- Comprehensive ERD showing all relationship types (1:1, 1:N, N:N) with complete attributes
- Example tables: users, profiles, posts, comments, roles, tags
- Junction tables: user_roles, post_tags with composite PKs
- All tables with PK (🔑) and FK (🔗) clearly marked
- Detailed indexes section

**These will be automatically populated in Step 7** when templates are copied and placeholders are replaced.

**This step extracts PRD-specific database tables** to be added to the Tables section in Step 7.8.

Read PRD and extract project-specific tables:

```
Extract from PRD:
  - All entities mentioned in System Modules
  - Relationships between entities
  - Special fields and constraints from PRD
```

**Store extracted PRD-specific table schemas in memory for Step 7.8**:

```markdown
### [table_name_from_PRD]

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| [column] | [type] | [Yes/No] | [default] | [description from PRD] |
| created_at | timestamp | No | now() | Creation time |
| updated_at | timestamp | No | now() | Last update |

**Constraints:**
- FOREIGN KEY [column] REFERENCES [table](id) ON DELETE [CASCADE/SET NULL]
- UNIQUE ([column])
```

**Extract project-specific entity relationships for ERD**:

```
[entity1] 1──N [entity2] ([relationship_description])
[entity3] N──N [entity4] (via [junction_table])
```

**Note**:
- The template ERD will serve as reference examples
- PRD-specific tables should be added to the Tables section (not replace examples)
- PRD-specific relationships should be added to Entity Relationships section
- The comprehensive ERD with all relationship types remains as documentation reference

#### 0.5.6.4 Cross-Check HTML Files with PRD

Before populating API integration documentation, cross-reference HTML files with PRD to ensure accuracy and identify any discrepancies.

**Cross-Check Process:**

```
1. List all HTML files from .claude-project/resources/HTML/
2. Extract screen names from PRD Part 2 & Part 3 (Page Architecture sections)
3. Match HTML files to PRD screens
4. Identify discrepancies:
   - HTML files not in PRD → mark as "Extra Screen" in docs
   - PRD screens without HTML → mark as "HTML Pending" in status
   - Name mismatches → use PRD naming, note HTML filename in parentheses
```

**Generate Cross-Check Table:**

```markdown
## HTML ↔ PRD Cross-Check

| HTML File | PRD Screen Match | PRD Section | Status |
|-----------|------------------|-------------|--------|
| 01-landing.html | Splash/Landing Page | Part 2: Common | Matched |
| 02-login.html | Login Page | Part 2: Common | Matched |
| 03-signup.html | Sign Up Page | Part 2: Common | Matched |
| 07-business-home.html | Home Tab - Dashboard Overview | Part 2: Business User | Matched |
| 14-dashboard-builder.html | Dashboard Builder Page | Part 2: Data Analyst | Matched |
| 18-ops-dashboard.html | Operations Dashboard Tab | Part 2: Ops Manager | Matched |
| 23-admin-dashboard.html | Dashboard Home Page | Part 3: Admin | Matched |
| custom-screen.html | - | - | Extra Screen |
| - | Mobile Settings Page | Part 2: Mobile | HTML Pending |
```

**Reconciliation Actions:**
- **Matched**: HTML file corresponds to PRD screen - document normally
- **Extra Screen**: HTML exists but not in PRD - add to PROJECT_API_INTEGRATION.md with note "Extra Screen - verify with design"
- **HTML Pending**: PRD screen exists but no HTML - mark status as "HTML Pending" in PROJECT_API_INTEGRATION.md
- **Name Mismatch**: Use PRD screen name as primary, note HTML filename in parentheses

#### 0.5.6.5 Update PROJECT_API_INTEGRATION.md

Map HTML screens to API endpoints:

```markdown
## Frontend Pages → API Mapping

### Auth Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| 01-landing.html | `/` | - | Pending |
| 02-login.html | `/login` | POST /auth/login | Pending |
| 03-signup.html | `/signup` | POST /auth/register | Pending |
| 04-forgot-password.html | `/forgot-password` | POST /auth/forgot-password | Pending |
| 05-reset-password.html | `/reset-password` | POST /auth/reset-password | Pending |
| 06-email-verification.html | `/verify-email` | POST /auth/verify-email | Pending |

### Business User Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| 07-business-home.html | `/home` | GET /dashboards/recent, GET /alerts/active | Pending |
| 08-dashboard-list.html | `/dashboards` | GET /dashboards | Pending |
| 09-dashboard-view.html | `/dashboards/:id` | GET /dashboards/:id, GET /widgets | Pending |
| 10-reports-list.html | `/reports` | GET /reports | Pending |
| 11-alerts-list.html | `/alerts` | GET /alerts | Pending |
| 12-user-settings.html | `/settings` | GET /users/me, PATCH /users/me | Pending |
| alert-details.html | `/alerts/:id` | GET /alerts/:id | Pending |
| modal-create-alert.html | - | POST /alerts | Pending |
| modal-schedule-report.html | - | POST /reports | Pending |

### Data Analyst Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| 13-analyst-home.html | `/analyst` | GET /dashboards, GET /models | Pending |
| 14-dashboard-builder.html | `/builder/:id` | GET/POST/PATCH dashboards, widgets | Pending |
| 15-query-editor.html | `/query` | POST /queries/execute, GET /queries/history | Pending |
| 16-data-models.html | `/models` | GET /models | Pending |
| 17-data-model-editor.html | `/models/:id` | GET/PATCH /models/:id | Pending |

### Operations Manager Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| 18-ops-dashboard.html | `/ops` | GET /ops/dashboard | Pending |
| 19-team-overview.html | `/team` | GET /ops/team | Pending |
| 20-team-member-detail.html | `/team/:id` | GET /ops/team/:id | Pending |
| 21-workflows.html | `/workflows` | GET /ops/workflows | Pending |
| 22-sla-monitor.html | `/sla` | GET /ops/sla | Pending |

### Admin Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| 23-admin-dashboard.html | `/admin` | GET /admin/stats | Pending |
| 24-user-management.html | `/admin/users` | GET /admin/users | Pending |
| user-details.html | `/admin/users/:id` | GET /admin/users/:id | Pending |
| create-user.html | `/admin/users/new` | POST /admin/users | Pending |
| 25-data-sources.html | `/admin/sources` | GET /data-sources | Pending |
| data-source-details.html | `/admin/sources/:id` | GET /data-sources/:id | Pending |
| add-connection.html | `/admin/sources/new` | POST /data-sources | Pending |
| 26-api-keys.html | `/admin/keys` | GET /admin/api-keys | Pending |
| api-key-details.html | `/admin/keys/:id` | GET /admin/api-keys/:id | Pending |
| generate-api-key.html | `/admin/keys/new` | POST /admin/api-keys | Pending |
| 27-branding.html | `/admin/branding` | GET/PATCH /admin/branding | Pending |
| 28-system-health.html | `/admin/health` | GET /admin/health | Pending |
| 29-audit-log.html | `/admin/audit` | GET /admin/audit-log | Pending |
| audit-log-details.html | `/admin/audit/:id` | GET /admin/audit-log/:id | Pending |
| 30-billing.html | `/admin/billing` | GET /billing/subscription | Pending |
| invoice-details.html | `/admin/billing/invoices/:id` | GET /billing/invoices/:id | Pending |

### Settings Pages

| HTML File | Route | API Endpoints | Status |
|-----------|-------|---------------|--------|
| settings-dashboard-prefs.html | `/settings/dashboard` | GET/PATCH /users/me/preferences | Pending |
| settings-notifications.html | `/settings/notifications` | GET/PATCH /users/me/notifications | Pending |
| settings-security.html | `/settings/security` | PATCH /users/me/password | Pending |
```

---

## Mode Branching

After completing Step 0.5 (resource detection and migration), the command branches based on the mode:

### If --docs-only Mode

1. **Verify Prerequisites:**
   ```bash
   if [ ! -d ".claude" ]; then
     echo "ERROR: Cannot use --docs-only without .claude/ submodule."
     echo "Run without --docs-only flag for full setup."
     exit 1
   fi
   ```

2. **Auto-detect Tech Stack from Existing Folders:**
   ```bash
   # Detect backend
   if [ -f "backend/package.json" ] && grep -q "@nestjs/core" backend/package.json; then
     BACKEND="nestjs"
   elif [ -f "backend/manage.py" ]; then
     BACKEND="django"
   fi

   # Detect frontends
   FRONTENDS=()
   [ -d "frontend" ] && FRONTENDS+=("react")
   [ -d "mobile" ] && FRONTENDS+=("react-native")

   # Detect dashboards
   DASHBOARDS=()
   [ -d "frontend-admin-dashboard" ] && DASHBOARDS+=("admin")
   [ -d "frontend-operations-dashboard" ] && DASHBOARDS+=("operations")
   [ -d "frontend-analytics-dashboard" ] && DASHBOARDS+=("analytics")
   [ -d "frontend-coach-dashboard" ] && DASHBOARDS+=("coach")

   # Also use values from Step 0.5.4 (PRD detection)
   ```

3. **Skip to Step 7:** Jump directly to "Step 7: Create Project Documentation Structure"

### If Normal Mode

1. **Continue to Step 1:** Proceed with "Step 1: Gather Tech Stack"
2. Use detected values from Step 0.5 as defaults in user prompts

---

## Step 1: Gather Tech Stack

Use **AskUserQuestion** to ask for the complete tech stack.

**If Step 0.5 detected tech stack values, use them as defaults (pre-selected).**

### Backend Framework (Required)

Default Selected: `$BACKEND_DETECTED` (if available from Step 0.5)

- **NestJS** (Recommended) - TypeScript, TypeORM, JWT, Swagger
- **Django** - Python, DRF, SimpleJWT, drf-spectacular

Store as:
- `$BACKEND` = "nestjs" | "django"

### Frontend Framework(s) (Required, multiSelect: true)

Default Selected: `$FRONTENDS_DETECTED` (if available from Step 0.5)

- **React Web** - React 19, TailwindCSS 4, shadcn/ui
- **React Native** - NativeWind, React Navigation, Detox

At least one must be selected. Both can be selected.

Store as:
- `$FRONTENDS` = array of ["react", "react-native"] (at least one required)

### Dashboard(s) (Optional, multiSelect: true)

Default Selected: `$DASHBOARDS_DETECTED` (if available from Step 0.5)

**What dashboards do you need?**

| Option | Description |
|--------|-------------|
| **Admin Dashboard** | Create `frontend-admin-dashboard/` for system administration |
| **Operations Dashboard** | Create `frontend-operations-dashboard/` for operations monitoring |
| **Analytics Dashboard** | Create `frontend-analytics-dashboard/` for data analytics |
| **Coach Dashboard** | Create `frontend-coach-dashboard/` for coach/user-specific features |
| **None** | No dashboard needed |

Note: Dashboards use React Web (Next.js). Can be selected regardless of frontend choice (e.g., React Native mobile + React dashboard).

Store as:
- `$DASHBOARDS` = array of ["admin", "operations", "analytics", "coach"] (can be empty or have multiple)

## Step 2: Confirm Project Structure

Display the planned structure and ask for confirmation:

```
=== Project Setup Plan: $PROJECT_NAME ===

Claude Configuration:
  - Uses shared claude-workflow submodule
  - Contains: base, nestjs, django, react, react-native skills

Boilerplate Code:
  - backend/                       ← nestjs-starter-kit (if NestJS)
  - backend/                       ← django-starter-kit (if Django)
  - frontend/                      ← react-starter-kit (if React Web)
  - frontend-admin-dashboard/      ← react-starter-kit (if Admin Dashboard)
  - frontend-operations-dashboard/ ← react-starter-kit (if Operations Dashboard)
  - frontend-analytics-dashboard/  ← react-starter-kit (if Analytics Dashboard)
  - frontend-coach-dashboard/      ← react-starter-kit (if Coach Dashboard)
  - mobile/                        ← react-native-starter-kit (if React Native)

Project Documentation:
  - .claude-project/status/
  - .claude-project/memory/
  - .claude-project/docs/

Proceed with this setup?
```

## Step 3: Create Project Directory

```bash
# Check if we're in an empty directory or need to create one
if [ "$(ls -A .)" ]; then
    mkdir $PROJECT_NAME
    cd $PROJECT_NAME
fi

# Initialize git if not already
git init 2>/dev/null || true
```

## Step 4: Add Claude Configuration

Add the shared claude-workflow as `.claude` submodule:

```bash
git submodule add https://github.com/potentialInc/claude-workflow.git .claude
git submodule update --init --recursive
```

This provides:
- `base/` - shared commands and skills
- `nestjs/` - NestJS backend skills
- `django/` - Django REST Framework skills
- `react/` - React web skills
- `react-native/` - React Native mobile skills

## Step 5: Clone Boilerplate Repositories

### Boilerplate Mapping

| Selection | Repository | Folder |
|-----------|------------|--------|
| NestJS | `https://github.com/potentialInc/nestjs-starter-kit` | `backend/` |
| Django | `https://github.com/potentialInc/django-starter-kit` | `backend/` |
| React Web | `https://github.com/potentialInc/react-starter-kit` | `frontend/` |
| Admin Dashboard | `https://github.com/potentialInc/react-starter-kit` | `frontend-admin-dashboard/` |
| Operations Dashboard | `https://github.com/potentialInc/react-starter-kit` | `frontend-operations-dashboard/` |
| Analytics Dashboard | `https://github.com/potentialInc/react-starter-kit` | `frontend-analytics-dashboard/` |
| Coach Dashboard | `https://github.com/potentialInc/react-starter-kit` | `frontend-coach-dashboard/` |
| React Native | `https://github.com/potentialInc/react-native-starter-kit` | `mobile/` |

### Clone Each Selected Repo

```bash
# For each selected boilerplate:
git clone --branch main --single-branch $REPO_URL $FOLDER

# Remove Docker files from subfolder
rm -f $FOLDER/Dockerfile* $FOLDER/docker-compose*.yml $FOLDER/.dockerignore

# Remove .git to maintain single history
rm -rf $FOLDER/.git $FOLDER/.gitmodules
```

## Step 6: Generate docker-compose.yml

Create root `docker-compose.yml` based on selected services:

```yaml
version: '3.8'

services:
  # Backend service (if selected)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-backend
    restart: unless-stopped
    ports:
      - '3000:3000'  # NestJS
      # - '8000:8000'  # Django
    environment:
      - NODE_ENV=${NODE_ENV:-development}
    networks:
      - $PROJECT_NAME-network

  # Frontend service (if selected)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-frontend
    restart: unless-stopped
    ports:
      - '5173:5173'
    networks:
      - $PROJECT_NAME-network

  # Admin Dashboard service (if selected)
  frontend-admin-dashboard:
    build:
      context: ./frontend-admin-dashboard
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-admin-dashboard
    restart: unless-stopped
    ports:
      - '5174:5173'
    networks:
      - $PROJECT_NAME-network

  # Operations Dashboard service (if selected)
  frontend-operations-dashboard:
    build:
      context: ./frontend-operations-dashboard
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-operations-dashboard
    restart: unless-stopped
    ports:
      - '5175:5173'
    networks:
      - $PROJECT_NAME-network

  # Analytics Dashboard service (if selected)
  frontend-analytics-dashboard:
    build:
      context: ./frontend-analytics-dashboard
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-analytics-dashboard
    restart: unless-stopped
    ports:
      - '5176:5173'
    networks:
      - $PROJECT_NAME-network

  # Coach Dashboard service (if selected)
  frontend-coach-dashboard:
    build:
      context: ./frontend-coach-dashboard
      dockerfile: Dockerfile
    container_name: $PROJECT_NAME-coach-dashboard
    restart: unless-stopped
    ports:
      - '5177:5173'
    networks:
      - $PROJECT_NAME-network

networks:
  $PROJECT_NAME-network:
    driver: bridge
```

## Step 7: Create Project Documentation Structure

This step can be run standalone with `--docs-only` flag for existing projects.

### 7.1 Check for existing .claude-project

```bash
if [ -d ".claude-project" ]; then
  # Ask user: Overwrite, Merge, or Skip?
  # - Overwrite: rm -rf .claude-project && continue
  # - Merge: continue (will skip existing files)
  # - Skip: exit step 7
fi
```

### 7.2 Copy templates

```bash
# Create .claude-project directory
mkdir -p .claude-project

# Copy all templates from .claude/templates
cp -r .claude/templates/claude-project/* .claude-project/
```

### 7.3 Rename template files (remove .template suffix)

```bash
# Find all .template.md files recursively and rename them
find .claude-project -name "*.template.md" | while read f; do
  mv "$f" "${f%.template.md}.md"
done
```

### 7.4 Replace placeholders in all markdown files

Templates use `{PLACEHOLDER}` format (curly braces).

```bash
# Replace {PROJECT_NAME} placeholder
find .claude-project -name "*.md" -exec sed -i '' "s/{PROJECT_NAME}/$PROJECT_NAME/g" {} \;

# Replace {BACKEND} placeholder
find .claude-project -name "*.md" -exec sed -i '' "s/{BACKEND}/$BACKEND/g" {} \;

# Replace {FRONTENDS} placeholder (join array with comma)
FRONTENDS_STR=$(IFS=', '; echo "${FRONTENDS[*]}")
find .claude-project -name "*.md" -exec sed -i '' "s/{FRONTENDS}/$FRONTENDS_STR/g" {} \;

# Replace {DATE} placeholder
find .claude-project -name "*.md" -exec sed -i '' "s/{DATE}/$(date +%Y-%m-%d)/g" {} \;
```

### 7.4.5 Filter Framework-Specific Sections Based on Tech Stack

**Purpose:** Automatically remove architecture sections for frameworks not in the selected tech stack.

**Logic:**
- Removes Django section if backend is NestJS
- Removes NestJS section if backend is Django
- Removes React Native section if not in frontends array
- Removes mobile/ directory reference if no React Native
- Removes React section if not in frontends array (edge case)

**Implementation:** Uses `sed` range deletion with conditional comment markers.

```bash
# Remove unused backend architecture sections
if [ "$BACKEND" = "nestjs" ]; then
  # Remove Django section
  sed -i '' '/<!-- Django Backend Architecture -->/,/<!-- End Django section -->/d' .claude-project/docs/PROJECT_KNOWLEDGE.md
elif [ "$BACKEND" = "django" ]; then
  # Remove NestJS section
  sed -i '' '/<!-- NestJS Backend Architecture -->/,/<!-- End NestJS section -->/d' .claude-project/docs/PROJECT_KNOWLEDGE.md
fi

# Remove React Native section if not in frontends
if [[ ! " ${FRONTENDS[@]} " =~ " react-native " ]]; then
  # Remove React Native Mobile Architecture section
  sed -i '' '/<!-- React Native Mobile Architecture -->/,/<!-- End React Native section -->/d' .claude-project/docs/PROJECT_KNOWLEDGE.md

  # Remove mobile directory line from Project Structure
  sed -i '' '/├── mobile\//d' .claude-project/docs/PROJECT_KNOWLEDGE.md
fi

# Remove React section if not in frontends (edge case)
if [[ ! " ${FRONTENDS[@]} " =~ " react " ]]; then
  # Remove React Web Architecture section
  sed -i '' '/<!-- React Web Architecture -->/,/<!-- End React section -->/d' .claude-project/docs/PROJECT_KNOWLEDGE.md
fi

echo "✓ Filtered framework-specific sections based on tech stack"
```

### 7.4.6 Remove Generic Database Template if PRD Exists

**Purpose:** Automatically remove generic ERD examples from PROJECT_DATABASE.md when a PRD file exists with database schema information.

**Logic:**
- Check if PRD file exists with database schema information
- If PRD exists: Remove generic template ERD section (will be replaced by PRD-specific schema in Step 7.8.2)
- If no PRD: Keep generic template ERD as reference for manual customization

**Implementation:** Uses `sed` range deletion with conditional comment markers.

```bash
# Check if PRD file exists with database schema information
if [ -f ".claude-project/prd/prd.pdf" ]; then
  echo "✓ PRD detected - will generate database schema from PRD entities"

  # Remove generic template ERD section
  sed -i '' '/<!-- Generic Template ERD -->/,/<!-- End Generic Template ERD -->/d' \
    .claude-project/docs/PROJECT_DATABASE.md

  # Note: PRD entity extraction and ERD generation happens in Step 7.8.2
else
  echo "⚠ No PRD found - keeping generic template ERD for reference"
fi
```

### 7.5 Handle gitignore template

```bash
# Append gitignore rules from template (if exists)
if [ -f ".claude-project/gitignore.template" ]; then
  cat .claude-project/gitignore.template >> .gitignore
  rm .claude-project/gitignore.template
fi
```

### 7.6 Create status folders for detected project folders

```bash
# Create status subfolders for each existing project folder
for folder in backend frontend frontend-admin-dashboard frontend-operations-dashboard frontend-analytics-dashboard frontend-coach-dashboard mobile; do
  if [ -d "$folder" ]; then
    mkdir -p ".claude-project/status/$folder"
    # Copy relevant status templates if they exist
    if [ -d ".claude-project/status/$folder" ]; then
      echo "Created status folder for $folder"
    fi
  fi
done
```

### 7.7 Verify documentation was created

```bash
# Verify key files exist and are not empty
required_files=(
  ".claude-project/docs/PROJECT_KNOWLEDGE.md"
  ".claude-project/docs/PROJECT_API.md"
  ".claude-project/memory/DECISIONS.md"
)

for file in "${required_files[@]}"; do
  if [ ! -s "$file" ]; then
    echo "WARNING: $file is missing or empty"
  fi
done
```

### 7.8 Populate PRD-Specific Content into Documentation

After templates are copied and placeholders replaced, add PRD-specific content extracted in Step 0.5.6.

#### 7.8.1 Update PROJECT_KNOWLEDGE.md with PRD Content

```bash
# Add PRD-specific sections to PROJECT_KNOWLEDGE.md
# The template already has Tech Stack and Architecture sections populated
# We only add PRD-specific content: Overview, Goals, User Types, Terminology, External Services

# Insert after "## Overview" placeholder (line ~3-4)
# - Add extracted project description

# Insert after "### Goals" placeholder
# - Add numbered goals from PRD

# Add User Types section after Tech Stack
# - Add extracted user types table

# Add Terminology section
# - Add extracted terminology table

# Add External Services section (after Architecture)
# - Add extracted external services table
```

**Note**: Do NOT modify the Architecture section - it's already comprehensive from the template and shows the correct framework based on {BACKEND} and {FRONTENDS} placeholders.

#### 7.8.2 Generate Database Schema from PRD

**Purpose:** If PRD exists, generate project-specific ERD and table schemas. Otherwise, keep generic template as reference.

**Note:** Generic ERD was already removed in Step 7.4.6 if PRD exists. This step generates the replacement schema.

```bash
if [ -f ".claude-project/prd/prd.pdf" ]; then

  echo "Generating database schema from PRD..."

  # Add project-specific ERD and schemas to PROJECT_DATABASE.md
  cat >> .claude-project/docs/PROJECT_DATABASE.md << 'DBEOF'

## Entity Relationship Diagram

### Application ERD

This ERD represents the application's database schema as specified in the PRD.

```
┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│            users                 │              │          dashboards              │
├──────────────────────────────────┤              ├──────────────────────────────────┤
│ 🔑 id (PK)          UUID         │──────1:N─────│ 🔑 id (PK)          UUID         │
│   email             VARCHAR(255) │   (creator)  │ 🔗 user_id (FK)     UUID         │
│   password          VARCHAR(255) │              │   name              VARCHAR(255) │
│   name              VARCHAR(100) │              │   description       TEXT         │
│   role              ENUM         │              │   layout            JSONB        │
│   created_at        TIMESTAMP    │              │   is_public         BOOLEAN      │
│   updated_at        TIMESTAMP    │              │   created_at        TIMESTAMP    │
└──────────────────────────────────┘              │   updated_at        TIMESTAMP    │
                │                                 └──────────────────────────────────┘
                │ 1:N                                            │
                │ (creator)                                      │ 1:N (parent)
                ▼                                                ▼
┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│            alerts                │              │           widgets                │
├──────────────────────────────────┤              ├──────────────────────────────────┤
│ 🔑 id (PK)          UUID         │              │ 🔑 id (PK)          UUID         │
│ 🔗 user_id (FK)     UUID         │              │ 🔗 dashboard_id (FK) UUID        │
│   name              VARCHAR(255) │              │ 🔗 data_source_id (FK) UUID      │
│   metric            VARCHAR(255) │              │   type              ENUM         │
│   threshold_type    ENUM         │              │   config            JSONB        │
│   threshold_value   DECIMAL      │              │   query             TEXT         │
│   status            ENUM         │              │   position          JSONB        │
│   created_at        TIMESTAMP    │              │   refresh_interval  INTEGER      │
│   updated_at        TIMESTAMP    │              │   created_at        TIMESTAMP    │
└──────────────────────────────────┘              │   updated_at        TIMESTAMP    │
                                                  └──────────────────────────────────┘

┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│         data_sources             │              │           reports                │
├──────────────────────────────────┤              ├──────────────────────────────────┤
│ 🔑 id (PK)          UUID         │              │ 🔑 id (PK)          UUID         │
│   name              VARCHAR(255) │              │ 🔗 dashboard_id (FK) UUID        │
│   type              ENUM         │              │   name              VARCHAR(255) │
│   connection_config JSONB        │              │   schedule          VARCHAR(50)  │
│   status            ENUM         │              │   format            ENUM         │
│   last_sync         TIMESTAMP    │              │   recipients        TEXT[]       │
│   created_at        TIMESTAMP    │              │   created_at        TIMESTAMP    │
└──────────────────────────────────┘              │   next_run          TIMESTAMP    │
                                                  └──────────────────────────────────┘

┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│         data_models              │              │          workflows               │
├──────────────────────────────────┤              ├──────────────────────────────────┤
│ 🔑 id (PK)          UUID         │              │ 🔑 id (PK)          UUID         │
│ 🔗 user_id (FK)     UUID         │              │ 🔗 requester_id (FK) UUID        │
│   name              VARCHAR(255) │              │ 🔗 approver_id (FK) UUID         │
│   query             TEXT         │              │   type              ENUM         │
│   tables            JSONB        │              │   status            ENUM         │
│   created_at        TIMESTAMP    │              │   request_data      JSONB        │
│   updated_at        TIMESTAMP    │              │   created_at        TIMESTAMP    │
└──────────────────────────────────┘              │   resolved_at       TIMESTAMP    │
                                                  └──────────────────────────────────┘

┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│       dashboard_shares           │              │          audit_logs              │
│      (Junction Table)            │              ├──────────────────────────────────┤
├──────────────────────────────────┤              │ 🔑 id (PK)          UUID         │
│ 🔑🔗 dashboard_id (PK,FK) UUID   │              │ 🔗 user_id (FK)     UUID         │
│ 🔑🔗 user_id (PK,FK)     UUID    │              │   action            VARCHAR(255) │
│ 🔗 shared_by (FK)        UUID    │              │   resource_type     VARCHAR(100) │
│   permission             ENUM    │              │   resource_id       UUID         │
│   created_at             TIMESTAMP│             │   details           JSONB        │
└──────────────────────────────────┘              │   ip_address        INET         │
                                                  │   created_at        TIMESTAMP    │
┌──────────────────────────────────┐              └──────────────────────────────────┘
│           api_keys               │
├──────────────────────────────────┤
│ 🔑 id (PK)          UUID         │
│ 🔗 user_id (FK)     UUID         │
│   key_hash          VARCHAR(255) │
│   name              VARCHAR(255) │
│   permissions       JSONB        │
│   last_used         TIMESTAMP    │
│   expires_at        TIMESTAMP    │
│   created_at        TIMESTAMP    │
└──────────────────────────────────┘

Legend:
🔑 Primary Key (PK)
🔗 Foreign Key (FK)
──  Relationship line
1:1 One-to-One relationship
1:N One-to-Many relationship
N:N Many-to-Many relationship (requires junction table with composite PK)
```

## Entity Relationships

### One-to-Many (1:N)

| Parent | Child | Relationship | FK Column | Constraint |
|--------|-------|--------------|-----------|------------|
| users | dashboards | One user creates many dashboards | dashboards.user_id → users.id | ON DELETE CASCADE |
| users | alerts | One user creates many alerts | alerts.user_id → users.id | ON DELETE CASCADE |
| users | data_models | One analyst creates many models | data_models.user_id → users.id | ON DELETE CASCADE |
| dashboards | widgets | One dashboard has many widgets | widgets.dashboard_id → dashboards.id | ON DELETE CASCADE |
| dashboards | reports | One dashboard has many reports | reports.dashboard_id → dashboards.id | ON DELETE CASCADE |
| data_sources | widgets | One data source used by many widgets | widgets.data_source_id → data_sources.id | ON DELETE SET NULL |
| users | workflows | One user initiates many workflows | workflows.requester_id → users.id | ON DELETE CASCADE |
| users | audit_logs | One user generates many audit logs | audit_logs.user_id → users.id | ON DELETE CASCADE |
| users | api_keys | One admin generates many API keys | api_keys.user_id → users.id | ON DELETE CASCADE |

### Many-to-Many (N:N)

| Entity 1 | Entity 2 | Junction Table | Composite PK | Description |
|----------|----------|----------------|--------------|-------------|
| dashboards | users | dashboard_shares | (dashboard_id, user_id) | Users can access shared dashboards |

## Tables

### users

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| email | VARCHAR(255) | No | - | Unique email address |
| password | VARCHAR(255) | No | - | Hashed password |
| name | VARCHAR(100) | Yes | NULL | Display name |
| role | ENUM | No | 'business_user' | business_user, analyst, ops_manager, admin |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update |

**Constraints:**
- UNIQUE (email)

### dashboards

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | FK to users.id (creator) |
| name | VARCHAR(255) | No | - | Dashboard name |
| description | TEXT | Yes | NULL | Dashboard description |
| layout | JSONB | No | '[]' | Widget layout configuration |
| is_public | BOOLEAN | No | false | Public visibility |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update |

**Constraints:**
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

### widgets

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| dashboard_id | UUID | No | - | FK to dashboards.id |
| data_source_id | UUID | Yes | NULL | FK to data_sources.id |
| type | ENUM | No | - | chart, table, metric, etc. |
| config | JSONB | No | '{}' | Widget configuration |
| query | TEXT | Yes | NULL | SQL query for data |
| position | JSONB | No | '{}' | Position and size |
| refresh_interval | INTEGER | Yes | NULL | Auto-refresh interval (seconds) |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update |

**Constraints:**
- FOREIGN KEY (dashboard_id) REFERENCES dashboards(id) ON DELETE CASCADE
- FOREIGN KEY (data_source_id) REFERENCES data_sources(id) ON DELETE SET NULL

### alerts

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | FK to users.id (creator) |
| name | VARCHAR(255) | No | - | Alert name |
| metric | VARCHAR(255) | No | - | Metric to monitor |
| threshold_type | ENUM | No | - | greater_than, less_than, equals |
| threshold_value | DECIMAL | No | - | Threshold value |
| status | ENUM | No | 'active' | active, triggered, snoozed, resolved |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update |

**Constraints:**
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

### data_sources

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| name | VARCHAR(255) | No | - | Data source name |
| type | ENUM | No | - | postgresql, mysql, bigquery, etc. |
| connection_config | JSONB | No | '{}' | Connection configuration (encrypted) |
| status | ENUM | No | 'pending' | pending, connected, error |
| last_sync | TIMESTAMP | Yes | NULL | Last metadata sync |
| created_at | TIMESTAMP | No | NOW() | Creation time |

**Constraints:**
- UNIQUE (name)

### reports

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| dashboard_id | UUID | No | - | FK to dashboards.id |
| name | VARCHAR(255) | No | - | Report name |
| schedule | VARCHAR(50) | No | - | Cron expression |
| format | ENUM | No | 'pdf' | pdf, csv, excel |
| recipients | TEXT[] | No | '{}' | Email recipients |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| next_run | TIMESTAMP | Yes | NULL | Next scheduled run |

**Constraints:**
- FOREIGN KEY (dashboard_id) REFERENCES dashboards(id) ON DELETE CASCADE

### data_models

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | FK to users.id (analyst) |
| name | VARCHAR(255) | No | - | Model name |
| query | TEXT | No | - | SQL query defining the model |
| tables | JSONB | No | '[]' | Referenced tables |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| updated_at | TIMESTAMP | No | NOW() | Last update |

**Constraints:**
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

### workflows

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| requester_id | UUID | No | - | FK to users.id |
| approver_id | UUID | Yes | NULL | FK to users.id |
| type | ENUM | No | - | dashboard_publish, data_source_add, etc. |
| status | ENUM | No | 'pending' | pending, approved, rejected |
| request_data | JSONB | No | '{}' | Workflow request data |
| created_at | TIMESTAMP | No | NOW() | Creation time |
| resolved_at | TIMESTAMP | Yes | NULL | Resolution time |

**Constraints:**
- FOREIGN KEY (requester_id) REFERENCES users(id) ON DELETE CASCADE
- FOREIGN KEY (approver_id) REFERENCES users(id) ON DELETE SET NULL

### dashboard_shares

Junction table for N:N relationship between dashboards and users.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| dashboard_id | UUID | No | - | FK to dashboards.id |
| user_id | UUID | No | - | FK to users.id |
| shared_by | UUID | No | - | FK to users.id (who shared) |
| permission | ENUM | No | 'view' | view, edit |
| created_at | TIMESTAMP | No | NOW() | Share time |

**Constraints:**
- Primary Key: (dashboard_id, user_id)
- FOREIGN KEY (dashboard_id) REFERENCES dashboards(id) ON DELETE CASCADE
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
- FOREIGN KEY (shared_by) REFERENCES users(id) ON DELETE CASCADE

### audit_logs

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | Yes | NULL | FK to users.id |
| action | VARCHAR(255) | No | - | Action performed |
| resource_type | VARCHAR(100) | No | - | Resource type (dashboard, user, etc.) |
| resource_id | UUID | Yes | NULL | Resource ID |
| details | JSONB | No | '{}' | Additional details |
| ip_address | INET | Yes | NULL | IP address |
| created_at | TIMESTAMP | No | NOW() | Action timestamp |

**Constraints:**
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE

### api_keys

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| user_id | UUID | No | - | FK to users.id (admin) |
| key_hash | VARCHAR(255) | No | - | Hashed API key |
| name | VARCHAR(255) | No | - | Key description |
| permissions | JSONB | No | '[]' | Permission scope |
| last_used | TIMESTAMP | Yes | NULL | Last usage timestamp |
| expires_at | TIMESTAMP | Yes | NULL | Expiration timestamp |
| created_at | TIMESTAMP | No | NOW() | Creation time |

**Constraints:**
- FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
- UNIQUE (key_hash)

DBEOF

  echo "✓ Generated database schema from PRD"

else
  echo "⚠ No PRD file found - using generic template"
fi
```

**Note:** Generic ERD from template is removed in Step 7.4.6 if PRD exists, then replaced with project-specific schema in this step.

### 7.9 Create CLAUDE.md

Create consolidated context file in root for token-efficient Claude interactions:

```bash
cat > CLAUDE.md << 'EOF'
# $PROJECT_NAME - Claude Context

## Stack
$BACKEND + ${FRONTENDS[*]} + PostgreSQL | Docker

## Architecture
```
backend/                    # $BACKEND API (port 3000)
${[[ " ${FRONTENDS[*]} " =~ " react " ]] && echo "frontend/                   # React Web (port 5173)"}
${[[ " ${DASHBOARDS[*]} " =~ " admin " ]] && echo "frontend-admin-dashboard/   # Admin (port 5174)"}
${[[ " ${DASHBOARDS[*]} " =~ " operations " ]] && echo "frontend-operations-dashboard/ # Operations (port 5175)"}
${[[ " ${DASHBOARDS[*]} " =~ " analytics " ]] && echo "frontend-analytics-dashboard/ # Analytics (port 5176)"}
${[[ " ${DASHBOARDS[*]} " =~ " coach " ]] && echo "frontend-coach-dashboard/   # Coach (port 5177)"}
${[[ " ${FRONTENDS[*]} " =~ " react-native " ]] && echo "mobile/                     # React Native"}
```

---

## Core BASH Tools (NO EXCEPTIONS)

```bash
# Pattern Search - USE 'rg' ONLY
rg -n "pattern" --glob '!node_modules/*'
rg -l "pattern"              # List matching files
rg -t py "pattern"           # Search Python files only

# File Finding - USE 'fd' ONLY
fd filename                  # Find by name
fd -e py                     # Find Python files
fd -H .env                   # Include hidden

# Bulk Operations - ONE command > many edits
rg -l "old" | xargs sed -i '' 's/old/new/g'

# Preview - USE 'bat'
bat -n filepath              # With line numbers
bat -r 10:50 file            # Lines 10-50

# JSON - USE 'jq'
jq '.dependencies | keys[]' package.json
```

---

## Commands (/slash)

| Category | Command | Purpose |
|----------|---------|---------|
| **Git** | /commit | Commit main project, create PR to dev |
| | /commit-all | Commit all including submodules |
| | /pull | Pull latest from dev |
| **Dev** | /new-project | Create new project with boilerplate |
| | /fix-ticket | Analyze and fix Notion ticket |
| | /fullstack | Run autonomous dev loops |
| **Design** | /prd-to-design-prompts | Convert PRD to Aura prompts |
| | /prompts-to-aura | Execute prompts on Aura.build |

---

## Agents

| Agent | Location | Trigger |
|-------|----------|---------|
| backend-developer | .claude/$BACKEND/agents/ | backend/ changes |
| frontend-developer | .claude/react/agents/ | frontend/ changes |
| design-qa-agent | .claude/react/agents/ | UI component work |

---

## Docs Reference

| Doc | Path |
|-----|------|
| Knowledge | .claude-project/docs/PROJECT_KNOWLEDGE.md |
| API | .claude-project/docs/PROJECT_API.md |
| Database | .claude-project/docs/PROJECT_DATABASE.md |
| Integration | .claude-project/docs/PROJECT_API_INTEGRATION.md |
| PRD | .claude-project/prd/prd.pdf |
| HTML Screens | .claude-project/resources/HTML/ |

---

## Framework Guides

| Framework | Path | Count |
|-----------|------|-------|
| $BACKEND | .claude/$BACKEND/guides/ | 20+ guides |
| React | .claude/react/guides/ | 22 guides |
| React Native | .claude/react-native/guides/ | 20 guides |
EOF
```

### 7.10 Create README.md

```bash
# Copy README template to root
cp .claude/templates/README.template.md README.md

# Replace placeholders in README.md
sed -i '' "s/{PROJECT_NAME}/$PROJECT_NAME/g" README.md
sed -i '' "s/{BACKEND}/$BACKEND/g" README.md
sed -i '' "s/{FRONTENDS}/$FRONTENDS_STR/g" README.md
```

## Step 8: Create .gitignore

```bash
cat > .gitignore << 'EOF'
# Node
**/node_modules/
**/dist/
**/build/

# Python
**/__pycache__/
**/*.pyc
**/.venv/

# Environment
**/.env
**/.env.local
**/.env.*.local

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store
Thumbs.db
EOF
```

## Step 9: Initial Commit

```bash
git add -A
git commit -m "$(cat <<'EOF'
feat: Initial $PROJECT_NAME project setup

- .claude/ submodule using shared claude-workflow
- Backend: $BACKEND
- Frontend: $FRONTENDS
- Docker orchestration configured

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## Step 10: Create GitHub Repo and Push

### 10.1 Create Main Project Repo on GitHub

```bash
gh repo create potentialInc/$PROJECT_NAME --private --description "$PROJECT_NAME - Full stack application"
```

### 10.2 Add Remote and Push main Branch

```bash
git remote add origin https://github.com/potentialInc/$PROJECT_NAME.git
git branch -M main
git push -u origin main
```

### 10.3 Create and Push dev Branch

```bash
git checkout -b dev
git push -u origin dev
```

### 10.4 Set dev as Default Branch (Optional)

```bash
# Set dev as the default branch for development workflow
gh repo edit potentialInc/$PROJECT_NAME --default-branch dev
```

### 10.5 Return to main Branch

```bash
git checkout main
```

## Step 11: Final Report

```
=== Project Setup Complete ===

$PROJECT_NAME/
├── .claude/                         # Shared claude-workflow
│   ├── base/                        → shared commands/skills
│   ├── nestjs/                      → NestJS backend skills
│   ├── django/                      → Django REST Framework skills
│   ├── react/                       → React web skills
│   └── react-native/                → React Native mobile skills
├── .claude-project/                 # Project docs
│   ├── docs/
│   ├── memory/
│   ├── prd/
│   │   └── prd.pdf                  # Migrated from root (if found)
│   ├── resources/
│   │   └── HTML/                    # Migrated from root (if found)
│   │       └── ($HTML_FILE_COUNT files)
│   └── status/
│       ├── backend/
│       ├── frontend/
│       ├── frontend-admin-dashboard/
│       ├── frontend-operations-dashboard/
│       ├── frontend-analytics-dashboard/
│       ├── frontend-coach-dashboard/
│       └── mobile/
├── backend/                         # $BACKEND boilerplate
├── frontend/                        # React Web (if selected)
├── frontend-admin-dashboard/        # Admin Dashboard (if selected)
├── frontend-operations-dashboard/   # Operations Dashboard (if selected)
├── frontend-analytics-dashboard/    # Analytics Dashboard (if selected)
├── frontend-coach-dashboard/        # Coach Dashboard (if selected)
├── mobile/                          # React Native (if selected)
├── docker-compose.yml
├── .gitignore
├── CLAUDE.md                        # Consolidated context for Claude
└── README.md

Migrated Resources (from Step 0.5):
  - HTML/ ($HTML_FILE_COUNT files) → .claude-project/resources/HTML/
  - prd.pdf → .claude-project/prd/prd.pdf

Auto-Detected Tech Stack (from PRD):
  - Backend: $BACKEND_DETECTED
  - Frontend: $FRONTENDS_DETECTED
  - Dashboards: $DASHBOARDS_DETECTED

GitHub Repo Created:
- https://github.com/potentialInc/$PROJECT_NAME (private)

Claude Configuration:
- Uses shared claude-workflow submodule
- https://github.com/potentialInc/claude-workflow

Branches Created:
- main (production-ready code)
- dev (active development)

Next Steps:
1. cd $PROJECT_NAME
2. docker-compose up -d
3. Begin development on dev branch!
```

## Error Handling

| Error | Resolution |
|-------|------------|
| `gh` not authenticated | Run `gh auth login` |
| Repo already exists | Ask to use existing or choose new name |
| Clone failed | Check network and repo access |
| Directory not empty | Ask to proceed or choose new location |

## Rollback

If setup fails midway:

```bash
# Clean up local
rm -rf backend frontend frontend-dashboard mobile .claude .claude-project docker-compose.yml

# Clean up GitHub (if repo was created)
gh repo delete potentialInc/$PROJECT_NAME --yes
```
