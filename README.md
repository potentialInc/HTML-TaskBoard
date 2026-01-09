# claude-base

Shared Claude Code configuration and commands for all projects.

---

## Quick Start (For Team Members)

### Prerequisites

1. **Install Claude Code**
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **Authenticate with GitHub CLI**
   ```bash
   gh auth login
   ```

3. **Authenticate with Claude**
   ```bash
   claude
   # Follow the authentication prompts
   ```

---

### Step-by-Step: Create a New Project

#### Step 1: Clone this repo (one-time setup)

```bash
git clone https://github.com/potentialInc/claude-base.git ~/claude-base
```

#### Step 2: Navigate to claude-base

```bash
cd ~/claude-base
```

#### Step 3: Start Claude Code

```bash
claude
```

#### Step 4: Run the new-project command

```
/new-project my-app-name
```

#### Step 5: Follow the prompts

The command will ask you to select:
- **Backend**: NestJS, Django, or None
- **Frontend**: React Web, React Native, or None
- **Dashboard**: Yes or No (if React Web selected)

#### Step 6: Wait for setup to complete

The command will:
1. Create a new GitHub repo for Claude configuration (`potentialInc/my-app-name-claude`)
2. Clone the selected boilerplate repos
3. Set up Docker configuration
4. Create project documentation structure

#### Step 7: Start developing

```bash
cd my-app-name
docker-compose up -d
claude
```

---

### Keeping Commands Updated

To get the latest commands:

```bash
cd ~/claude-base
git pull origin main
```

---

## Available Commands

| Command | Description |
|---------|-------------|
| `/new-project` | Create a complete new project with Claude config and boilerplate |
| `/commit` | Smart git commit with branch management |
| `/pull` | Pull latest changes for all submodules |
| `/initialize-claude` | Initialize Claude Code configuration only (advanced) |
| `/create-mono-repo` | Clone boilerplate repos into mono-repo structure (advanced) |
| `/pdf-to-prd` | Convert PRD PDF file to structured markdown |
| `/extract-screens-from-figma` | Extract Figma screen names and node IDs |
| `/dev-docs` | Create strategic plan with task breakdown |
| `/dev-docs-update` | Update dev documentation before context compaction |
| `/ralph` | Run autonomous workflow loops (design-qa, e2e-tests, backend-qa, api-docs) |

---

## Project Structure After `/new-project`

```
my-app-name/
├── .claude/                    # Claude Code configuration (submodule)
│   ├── base/                   → claude-base (this repo)
│   ├── nestjs/                 → claude-nestjs (if NestJS selected)
│   ├── react/                  → claude-react (if React selected)
│   ├── commands -> base/commands
│   └── settings.json
├── .claude-project/            # Project documentation
│   ├── docs/                   # Technical documentation
│   ├── plans/                  # Implementation plans
│   ├── memory/                 # Context for Claude
│   ├── prd/                    # Product requirements
│   └── secrets/                # Credentials (gitignored)
├── backend/                    # NestJS or Django (if selected)
├── frontend/                   # React Web (if selected)
├── frontend-dashboard/         # Admin dashboard (if selected)
├── mobile/                     # React Native (if selected)
├── docker-compose.yml
├── .gitignore
└── README.md
```

---

## Troubleshooting

### "Command not found" when running /new-project

**Cause**: You're not in a directory with `.claude/` folder or Claude can't find the commands.

**Solution**: Make sure you're in the `claude-base` directory:
```bash
cd ~/claude-base
claude
/new-project my-app
```

### "gh: command not found"

**Cause**: GitHub CLI is not installed.

**Solution**:
```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Then authenticate
gh auth login
```

### "Repository already exists"

**Cause**: A repo with that name already exists on GitHub.

**Solution**: Choose a different project name or delete the existing repo:
```bash
gh repo delete potentialInc/my-app-name-claude --yes
```

### Submodule issues after cloning a project

**Solution**:
```bash
git submodule update --init --recursive
```

---

## For Maintainers

### Repository Structure

```
claude-base/
├── agents/           # Reusable agent definitions
├── commands/         # Slash commands
├── docs/             # Generic documentation
├── hooks/            # Git and Claude hooks
├── skills/           # Shared skills
└── templates/        # Project templates
    └── claude-project/  # .claude-project folder templates
```

### Related Repositories

#### Claude Config Repos (Used as submodules)

| Repo | Purpose |
|------|---------|
| [claude-base](https://github.com/potentialInc/claude-base) | Shared base (this repo) |
| [claude-nestjs](https://github.com/potentialInc/claude-nestjs) | NestJS patterns |
| [claude-django](https://github.com/potentialInc/claude-django) | Django patterns |
| [claude-react](https://github.com/potentialInc/claude-react) | React patterns |
| [claude-react-native](https://github.com/potentialInc/claude-react-native) | React Native patterns |

#### Boilerplate Repos (Cloned by /new-project)

| Repo | Purpose |
|------|---------|
| [nestjs-starter-kit](https://github.com/potentialInc/nestjs-starter-kit) | NestJS backend |
| [django-starter-kit](https://github.com/potentialInc/django-starter-kit) | Django backend |
| [react-starter-kit](https://github.com/potentialInc/react-starter-kit) | React frontend |
| [react-native-starter-kit](https://github.com/potentialInc/react-native-starter-kit) | React Native mobile |

### Updating Commands

1. Edit files in `commands/` directory
2. Commit and push to this repo
3. All team members get updates via `git pull`

### Template Variables

Templates in `templates/claude-project/` use these placeholders:

| Variable | Description |
|----------|-------------|
| `$PROJECT_NAME` | Project name (lowercase, hyphenated) |
| `$BACKEND` | Selected backend framework |
| `$FRONTENDS` | Selected frontend frameworks |
