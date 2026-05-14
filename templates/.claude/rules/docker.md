---
paths:
  - "infrastructure/**"
  - "docker-compose*.yml"
  - "**/Dockerfile"
---

# Docker and Infrastructure Rules

Read `.claude/references/docker-services.md` for project-specific service map and ports.

## Critical Rules

- Always run `docker info > /dev/null 2>&1` preflight before any docker command
- If Docker is not running — stop and ask user to start it, do not retry
- Never run `docker compose down -v` without explicit user confirmation
- Confirm before: `docker volume rm`, `docker system prune`, `docker image prune`
- Never rebuild database containers without confirming data loss is acceptable

## Port Map

| Service | Port |
|---------|------|
| Engine | 8081 |
| HR | 8082 |
| Admin | 8083 |
| Finance | 8084 |
| Procurement | 8085 |
| Inventory | 8086 |
| Keycloak | 8090 |
| PostgreSQL | 5433 |
| pgAdmin | 5050 |
| Admin Portal | 4000 |
| HR Portal | 4001 |
