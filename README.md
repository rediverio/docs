# ReDiver - CTEM Platform

**Continuous Threat Exposure Management Platform**

ReDiver giúp tổ chức quản lý rủi ro bảo mật thông qua quy trình CTEM 5 giai đoạn:
**Scoping** → **Discovery** → **Prioritization** → **Validation** → **Mobilization**

---

## Architecture

| Service | Description | Tech Stack |
|---------|-------------|------------|
| **rediver-api** | Backend REST API | Go 1.25, Chi, PostgreSQL 17, Redis 7 |
| **rediver-ui** | Frontend Application | Next.js 16, React 19, TypeScript, Tailwind 4 |
| **rediver-keycloak** | Identity Provider (Optional) | Keycloak 24+ |

---

## Documentation

### Getting Started
| Document | Description |
|----------|-------------|
| [Getting Started](./getting-started.md) | Quick start guide (10 min) |
| [Development Setup](./development-setup.md) | IDE, debugging, testing |
| [Environment Configuration](./environment-config.md) | All environment variables |
| [Troubleshooting](./troubleshooting.md) | Common issues and solutions |

### Authentication & Security
| Document | Description |
|----------|-------------|
| [Authentication Guide](./authentication-guide.md) | Login flow, JWT, sessions |
| [Multi-Tenancy Guide](./multi-tenancy-guide.md) | Teams, tenant switching |
| [Permissions Matrix](./permissions-matrix.md) | Role-based access control |

### Architecture & API
| Document | Description |
|----------|-------------|
| [Architecture](./architecture.md) | System design overview |
| [API Reference](./api-reference.md) | Complete API endpoints |

---

## Quick Start

### Prerequisites

| Tool | Version | Check |
|------|---------|-------|
| Docker | 24+ | `docker -v` |
| Node.js | 20+ | `node -v` |
| Go | 1.25+ | `go version` |

### Start with Docker

```bash
git clone https://github.com/rediverio/rediver.git
cd rediver

# Configure environment
cd rediver-api && cp .env.example .env && cd ..
cd rediver-ui && cp .env.example .env.local && cd ..

# Start all services
docker compose up -d
```

### Verify

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8080 |
| API Docs | http://localhost:8080/docs |

---

## CTEM 5-Stage Process

```
┌─────────────────────────────────────────────────────────────────┐
│  1. SCOPING     │  2. DISCOVERY   │  3. PRIORITIZATION          │
│  Define attack  │  Identify       │  Rank exposures by          │
│  surface scope  │  assets, repos  │  severity, SLA, risk        │
├─────────────────┴─────────────────┴─────────────────────────────┤
│  4. VALIDATION                    │  5. MOBILIZATION            │
│  Verify exploitability,           │  Remediation workflows,     │
│  pen testing, scans               │  assign, track, resolve     │
└───────────────────────────────────┴─────────────────────────────┘
```

---

## Multi-Tenancy

- **Tenant** (API) = **Team** (UI)
- Users can belong to multiple teams
- Data isolation per tenant
- Role-based permissions: **Owner > Admin > Member > Viewer**

```
User → Login → Select Team → Access Token (scoped to team) → API
```

---

## Service Ports

| Service | Port |
|---------|------|
| Frontend | 3000 |
| Backend API | 8080 |
| Keycloak | 8180 |
| PostgreSQL | 5432 |
| Redis | 6379 |

---

## Key Commands

### Backend
```bash
make dev              # Run with hot reload
make test             # Run tests
make migrate-up       # Database migrations
make lint             # Code linting
```

### Frontend
```bash
npm run dev           # Development server
npm run build         # Production build
npm run test          # Run tests
npm run lint          # ESLint
```

---

## Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/name`
3. Commit with conventional commits: `feat:`, `fix:`, `docs:`
4. Push and open Pull Request

---

## License

MIT License - see [LICENSE](../rediver-api/LICENSE)
