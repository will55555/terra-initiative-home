# Terra initiative Ecosystem

Multi-service orchestration for the Terra Inc ecosystem. Contains backend API, frontend dashboard, and Jenkins CI/CD infrastructure.

## Project Structure

```
terra-initiative-home/
├── docker-compose.yml          ← Single source of truth for local dev (full stack)
├── docker.env                  ← Local dev environment variables (gitignored)
├── .gitignore                  ← Prevents docker-compose files in individual repos
├── README.md                   ← This file
│
├── terra-api/                  ← Backend API (Spring Boot, Java 21)
│   ├── Dockerfile
│   ├── docker-compose.prod.yml ← EC2 production deployment (not local dev)
│   ├── docker-compose.staging.yml ← EC2 staging deployment (not local dev)
│   ├── build.gradle.kts
│   └── src/
│
├── terra-api-fe/               ← Frontend Dashboard (React 19, Redux)
│   ├── Dockerfile
│   ├── package.json
│   └── src/
│
├── terra-jenkins/              ← CI/CD Infrastructure (Jenkins)
│   ├── docker-compose.jenkins.yml ← Jenkins server (separate from app stack)
│   ├── jenkins.Dockerfile
│   └── Jenkinsfile
│
└── .github/                    ← GitHub Actions / repo config
```

## Local Development

### Quick Start

```bash
# From this folder (terra-api/)
docker-compose up --build

# Services will be available at:
# Backend API: http://localhost:8080 (changed from 8081, 2026-08-08)
# Frontend: http://localhost:3000
# Postgres: localhost:5433
# Redis: localhost:6379
```

### Individual Service Development

If you only want to run the backend or frontend:

```bash
# Backend only
cd terra-api
./gradlew bootRun

# Frontend only
cd terra-api-fe
npm start
```

## Important Notes

**Local-dev orchestration is composed at the root via `include:`**, not duplicated. The
root `docker-compose.yml` pulls in each service's own compose file (`terra-api/docker-compose.yml`,
`terra-api/docker-compose.dev.yml`, `terra-jenkins/docker-compose.jenkins.yml`) — it does not
redefine services itself.

- **Individual repos DO keep their own compose files** — that's required, not a drift risk: `terra-api/docker-compose.prod.yml` / `docker-compose.staging.yml` are EC2-specific deploys that have nothing to do with local dev, and `terra-api/docker-compose.dev.yml` / `terra-jenkins/docker-compose.jenkins.yml` are exactly what this root file includes.
- **Only `docker.env` (parent-level, gitignored) and this root `docker-compose.yml` are orchestration-specific to this repo** — everything else lives with its own service.

## Architecture

### Phases

| Phase | Status | Features |
|---|---|---|
| 1–3 | Complete | JWT auth, rate limiting, audit logs, resilience |
| 4 | Complete | Feature flags, ecosystem health orchestration |
| 5 | Complete | Redis + Postgres migrations |
| 6 | Live | Jenkins CI/CD pipeline, EC2 deployment |

See [ADRs in Notion](https://notion.so/terra-api-architecture) for detailed decision records (ADR-001 through ADR-010).

### Deployment

**Local:** `docker-compose up` (this folder)  
**EC2 Staging:** `docker-compose-v2 -p terra-api-staging -f docker-compose.staging.yml up -d` (separate port 9081)  
**EC2 Prod:** `docker-compose-v2 -p terra-api-prod -f docker-compose.prod.yml up -d` (port 8080, changed from 8081 2026-08-08; also reachable via HTTPS at `api.terra-hq.com`)

Both tiers run on the same EC2 box, kept separate via Compose project names.

## Remotes

Both repos sync to GitHub + Bitbucket mirror:

```bash
git push all --all  # Pushes to both remotes simultaneously
```

See memory at `~/.claude/projects/c--Users-test-Desktop-Programing-terra-api-home/memory/` for multi-remote workflow (path is machine-specific — verify on any new machine).

## Development Workflow

1. Clone this parent folder
2. Individual repos stay nested (terra-api, terra-api-fe, etc.)
3. Run `docker-compose up` from this folder for full-stack dev
4. Commit work in individual repos; they have separate remotes
5. Jenkins auto-builds on push (feature branches CI-only, `phase-*` staging manual-approval, `master` prod auto)

## Troubleshooting

**"docker-compose: command not found"** → Install Docker Desktop or `docker-compose` CLI

**Port already in use** → Check `docker ps` for existing containers; stop with `docker-compose down`

**Postgres/Redis not healthy** → Logs: `docker-compose logs postgres` / `docker-compose logs redis`

## Contact

See `TASKS.md` for active work; `DEV_LOG.md` in terra-api/ for detailed history.
