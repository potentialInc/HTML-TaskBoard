# {PROJECT_NAME}

## Quick Start

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# Or if already cloned
git submodule update --init --recursive

# Start services
docker-compose up -d
```

## Documentation

- **Project Knowledge**: [.claude-project/docs/PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md)
- **API Reference**: [.claude-project/docs/PROJECT_API.md](.claude-project/docs/PROJECT_API.md)
- **Database Schema**: [.claude-project/docs/PROJECT_DATABASE.md](.claude-project/docs/PROJECT_DATABASE.md)
- **API Integration**: [.claude-project/docs/PROJECT_API_INTEGRATION.md](.claude-project/docs/PROJECT_API_INTEGRATION.md)

## Tech Stack

- **Backend**: {BACKEND}
- **Frontend**: {FRONTENDS}
- **Database**: PostgreSQL
- **Deployment**: Docker

## Project Structure

```
{PROJECT_NAME}/
├── .claude/                       # Claude Code configuration (submodule)
├── .claude-project/               # Project documentation & resources
│   ├── docs/                      # Technical documentation
│   ├── prd/                       # Product requirements
│   ├── resources/                 # HTML screens & assets
│   └── memory/                    # Project decisions & context
├── backend/                       # {BACKEND} API server
├── frontend/                      # React web application (if applicable)
├── frontend-admin-dashboard/      # Admin dashboard (if applicable)
├── frontend-operations-dashboard/ # Operations dashboard (if applicable)
├── frontend-analytics-dashboard/  # Analytics dashboard (if applicable)
├── frontend-coach-dashboard/      # Coach dashboard (if applicable)
├── mobile/                        # React Native mobile app (if applicable)
├── docker-compose.yml             # Container orchestration
├── .gitignore                     # Git ignore rules
├── CLAUDE.md                      # Consolidated context for Claude
└── README.md                      # This file
```

## Development

### Prerequisites

- Node.js 18+
- Docker & Docker Compose
- Git

### Initial Setup

```bash
# Install dependencies
cd backend && npm install
cd ../frontend && npm install

# Set up environment variables
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Start database
docker-compose up -d postgres

# Run migrations
cd backend && npm run migration:run

# Start development servers
npm run dev
```

### Running Tests

```bash
# Backend tests
cd backend && npm test

# Frontend tests
cd frontend && npm test
```

## Deployment

See [.claude-project/docs/PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md) for deployment instructions.

## Contributing

1. Create a feature branch from `dev`
2. Make your changes
3. Run tests
4. Create a pull request to `dev`

## License

[Add your license here]
