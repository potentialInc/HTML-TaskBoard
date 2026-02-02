---
description: Run tests with optional analysis, generation, and status update
argument-hint: "[target] [--full|--analyze|--generate|--run|--update] [--cleanup]"
---

# Test Workflow Command

A unified command for running tests across all project targets (backend, frontend, frontend-dashboard).

---

## Quick Reference

```bash
/test                      # Interactive mode (ask target & mode)
/test frontend             # Test frontend (ask mode)
/test all --full           # Full pipeline for all targets
/test backend --run        # Run backend tests only
```

---

## Step 1: Parse Arguments

```
$ARGUMENTS
```

**Extract:**
- `target`: frontend | backend | frontend-dashboard | all | (empty)
- `mode`: --full | --analyze | --generate | --run | --update | (empty)
- `options`: --cleanup (stop servers after tests)

---

## Step 2: Determine Target

### If target is NOT specified:

Use **AskUserQuestion** to ask:

```
Question: "Which project would you like to test?"
Header: "Target"
Options:
  1. "frontend" - Patient/Coach app
  2. "backend" - API server
  3. "frontend-dashboard" - Admin dashboard
  4. "all" - All of the above
```

### If target is specified:

Validate target exists:
- Check if directory exists: `{target}/`
- Check if test directory exists: `{target}/test/`

If invalid, show error and list available targets.

---

## Step 3: Check TESTCASE_STATUS.md

For each target, check:
```
.claude-project/status/{target}/TESTCASE_STATUS.md
```

### If file does NOT exist:

Use **AskUserQuestion**:

```
Question: "TESTCASE_STATUS.md not found for {target}. Create it now?"
Header: "Setup"
Options:
  1. "Yes" - Run /generate-testcase first
  2. "No" - Cancel
```

If "Yes": Execute `/generate-testcase {target}` then continue.

If "No": Stop execution.

---

## Step 4: Determine Mode

### If mode flag is specified:

| Flag | Execute |
|------|---------|
| `--full` | analyze → generate → run → update |
| `--analyze` | analyze only |
| `--generate` | analyze → generate |
| `--run` | run → update |
| `--update` | update only |

### If mode is NOT specified:

Use **AskUserQuestion**:

```
Question: "Where would you like to start?"
Header: "Mode"
Options:
  1. "From the beginning" - Test case generation → Analysis → Code generation → Run → Update
  2. "From analysis" - Missing test check → Code generation → Run → Update
  3. "From code generation" - Test code generation → Run → Update
  4. "Run tests only" - Run → Update
  5. "Update results only" - Update TESTCASE_STATUS.md with previous results
```

---

## Step 4.5: Start Required Servers (CRITICAL)

**Before running tests, ensure all required servers are running.**

### 4.5.1 Detect Server Configuration

Read server configuration from each target's package.json or config files:

```
For each target directory:
  1. Read package.json scripts (dev, start, start:dev)
  2. Read .env or config for PORT
  3. Check playwright.config.ts for baseURL
  4. Check vite.config.ts for server.port
```

**Auto-detect patterns:**

| Source | Extract |
|--------|---------|
| `package.json` scripts | `dev`, `start`, `start:dev` commands |
| `.env` / `.env.local` | `PORT`, `VITE_PORT`, `API_URL` |
| `playwright.config.ts` | `baseURL` in `use` config |
| `vite.config.ts` | `server.port` |
| `nest-cli.json` or similar | Backend port configuration |

### 4.5.2 Determine Required Servers

Based on target type:

| Target Type | Required Servers |
|-------------|------------------|
| Backend (API) | None for E2E (uses test DB) |
| Frontend (Web) | Backend API + Frontend dev server |
| Any with `baseURL` in playwright config | Parse URL to determine dependencies |

**Detection logic:**

```
For target:
  Read playwright.config.ts
  Extract baseURL (e.g., http://localhost:5173)
  Extract API URL from env or fixtures

  Required servers = [baseURL host:port]
  If tests call API: Add backend server
```

### 4.5.3 Server Health Check

Generic health check for any server:

```bash
# Check if port is in use
check_port() {
  lsof -i :$1 | grep LISTEN > /dev/null 2>&1
}

# Check if HTTP server responds
check_http() {
  curl -s --max-time 2 "$1" > /dev/null 2>&1
}
```

### 4.5.4 Auto-Start Servers

**Generic server startup flow:**

```
For each required server:
  1. Check if already running (port check)
  2. If not running:
     a. cd to target directory
     b. Run appropriate npm script in background
     c. Wait for server to be ready (with timeout)
  3. If timeout: Report warning, continue
```

**Startup command detection:**

```
Read package.json scripts:
  - Prefer: "start:dev" or "dev" for development
  - Fallback: "start"
  - For tests: Check for "test:e2e:server" or similar
```

**Example execution:**

```bash
# Generic server start
start_server() {
  local dir=$1
  local port=$2
  local timeout=${3:-60}

  if check_port $port; then
    echo "Server already running on port $port"
    return 0
  fi

  cd "$dir"

  # Detect start command from package.json
  if npm run | grep -q "start:dev"; then
    npm run start:dev &
  elif npm run | grep -q "dev"; then
    npm run dev &
  else
    npm start &
  fi

  # Wait for server
  for i in $(seq 1 $timeout); do
    check_port $port && return 0
    sleep 1
  done

  echo "Warning: Server startup timeout on port $port"
  return 1
}
```

### 4.5.5 Server Startup Order

```
1. Identify all required servers and their dependencies
2. Build dependency graph (e.g., frontend depends on backend)
3. Start in order: independent servers first, then dependents
4. Wait for each to be ready before starting dependents
```

### 4.5.6 Execution Flow

```
servers_needed = detect_required_servers(target)
servers_running = []

For server in servers_needed:
  If not is_running(server.port):
    start_server(server.directory, server.port)
    wait_for_ready(server.url, timeout=60)
  servers_running.append(server)

If all servers ready:
  Execute test workflow
Else:
  Report which servers failed
  Ask user to continue or abort
```

### 4.5.7 Cleanup (Optional)

Track started servers for optional cleanup after tests:

```
started_pids = []

# After tests complete:
If user requested cleanup:
  For pid in started_pids:
    kill pid
```

**Note:** By default, servers continue running after tests. Use `--cleanup` flag to stop them.

---

## Step 5: Execute Skills

Based on selected mode, execute skills in order:

### Mode: From the beginning (--full)

```
1. Run: /generate-testcase {target}
2. Load skill: .claude/base/skills/test/analyze.md
   Execute with target={target}
3. Load skill: .claude/base/skills/test/generate.md
   Execute with target={target}
4. Load skill: .claude/base/skills/test/run.md
   Execute with target={target}
5. Load skill: .claude/base/skills/test/update.md
   Execute with target={target}
```

### Mode: From analysis (--analyze + continue)

```
1. Load skill: .claude/base/skills/test/analyze.md
2. Load skill: .claude/base/skills/test/generate.md
3. Load skill: .claude/base/skills/test/run.md
4. Load skill: .claude/base/skills/test/update.md
```

### Mode: From code generation (--generate + continue)

```
1. Load skill: .claude/base/skills/test/generate.md
2. Load skill: .claude/base/skills/test/run.md
3. Load skill: .claude/base/skills/test/update.md
```

### Mode: Run tests only (--run)

```
1. Load skill: .claude/base/skills/test/run.md
2. Load skill: .claude/base/skills/test/update.md
```

### Mode: Update results only (--update)

```
1. Load skill: .claude/base/skills/test/update.md
```

---

## Step 6: Handle Multiple Targets

When `target = all`:

1. Detect all test targets in project root:
   - Scan for directories with `test/` subdirectory
   - Common patterns: `backend/`, `frontend*/`

2. Execute selected mode for each target sequentially:
   ```
   For each target in [backend, frontend, frontend-dashboard]:
     Execute mode workflow
     Report results
   ```

3. Show combined summary at the end.

---

## Step 7: Report Results

After execution, display summary:

```markdown
## Test Results Summary

| Target | Tests | Passed | Failed | Skipped | Duration |
|--------|-------|--------|--------|---------|----------|
| frontend | 76 | 74 | 2 | 0 | 45s |
| backend | 103 | 103 | 0 | 0 | 32s |
| frontend-dashboard | 50 | 48 | 1 | 1 | 28s |
| **Total** | 229 | 225 | 3 | 1 | 105s |

### Failed Tests

- frontend: `login.spec.ts:67` - should redirect coach to dashboard
- frontend: `home.spec.ts:45` - should display greeting
- frontend-dashboard: `users.spec.ts:89` - should filter by role

### Updated Files

- .claude-project/status/frontend/TESTCASE_STATUS.md
- .claude-project/status/backend/TESTCASE_STATUS.md
- .claude-project/status/frontend-dashboard/TESTCASE_STATUS.md
```

---

## Error Handling

| Error | Action |
|-------|--------|
| Target directory not found | Show available targets, ask to select |
| Test directory not found | Suggest creating test structure |
| TESTCASE_STATUS.md not found | Offer to run /generate-testcase |
| Test execution fails | Continue to update with failure results |
| Server not running (frontend) | Show instructions to start servers |

---

## Server Requirements

Server requirements are auto-detected from project configuration:

### Detection Sources

| File | Information |
|------|-------------|
| `playwright.config.ts` | `baseURL` for test target |
| `package.json` | `scripts.dev`, `scripts.start` |
| `.env` / `.env.local` | `PORT`, `API_URL`, `VITE_API_URL` |
| `vite.config.ts` | `server.port`, `server.proxy` |

### Common Patterns

| Target Type | Typical Requirements |
|-------------|---------------------|
| Backend E2E | Test database only (no live servers) |
| Frontend E2E | Backend API + Frontend dev server |
| Full-stack | All application servers |

### Manual Override

If auto-detection fails, create `.claude-project/config/test-servers.json`:

```json
{
  "backend": {
    "port": 4000,
    "startCommand": "npm run start:dev",
    "healthCheck": "/api-json"
  },
  "frontend": {
    "port": 5173,
    "startCommand": "npm run dev",
    "dependsOn": ["backend"]
  }
}
```

---

## Examples

### First time setup
```bash
/test frontend
# → TESTCASE_STATUS.md not found
# → "Create it now?" → Yes
# → "Where to start?" → From the beginning
# → Auto-detects and starts required servers
# → Full pipeline executes
```

### Daily testing
```bash
/test frontend --run
# → Checks server status
# → Starts missing servers automatically
# → Runs tests
# → Updates TESTCASE_STATUS.md
```

### After adding new feature
```bash
/test frontend
# → "Where to start?" → From analysis
# → Finds missing tests for new feature
# → Generates test code
# → Ensures servers are running
# → Runs all tests
# → Updates status
```

### CI/CD pipeline
```bash
/test all --run
# → Auto-starts all required servers
# → Runs all tests for all targets
# → No questions asked
# → Returns exit code based on results
```

### With server cleanup
```bash
/test all --run --cleanup
# → Starts servers
# → Runs tests
# → Stops servers that were started by this command
```
