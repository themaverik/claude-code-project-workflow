# Docker Services -- [Project Name]

Project-specific Docker configuration. Read by the global docker-dev skill when working in this project.

<!-- Customize everything below for your project -->

## Compose Files

| File | Purpose | Services |
|------|---------|----------|
| `docker-compose.yml` | Full stack | All services |
| `docker-compose.dev.yml` | Local dev | Infrastructure only |

Base path: `./` <!-- or infrastructure/docker/ etc. -->

## Service Map

| Service | Container | Port | Health Endpoint |
|---------|-----------|------|-----------------|
| PostgreSQL | project-postgres | 5432 | `pg_isready` |
| Redis | project-redis | 6379 | `redis-cli ping` |
| App | project-app | 3000 | `/health` |

<!-- Add all services from your docker-compose files -->

## Workflows

### Start dev environment

```bash
docker compose -f docker-compose.dev.yml up -d
```

### Start full stack

```bash
docker compose up -d --build
```

### Check status

```bash
docker compose ps
```

### Health check

```bash
curl -sf http://localhost:3000/health | head -1
```

### Database access

```bash
docker exec -it project-postgres psql -U <user> -d <database>
```

## Project Rules

1. Always specify the correct compose file when multiple exist
2. When the user says "start docker" without specifying, ask which environment
