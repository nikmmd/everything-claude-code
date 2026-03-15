---
name: docker-patterns
description: Docker best practices for multi-stage builds, layer caching, security hardening, and compose patterns. Auto-triggers when working with Dockerfiles, docker-compose, or container configuration.
compatibility: Designed for Claude Code
---

# Docker Patterns

Auto-activates when working with Dockerfiles, docker-compose, or container configuration.

## Multi-Stage Builds

Separate build dependencies from runtime:

```dockerfile
# Build stage
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json bun.lockb ./
RUN npm ci --production=false
COPY . .
RUN npm run build

# Runtime stage
FROM node:22-alpine AS runtime
WORKDIR /app
RUN addgroup -g 1001 appgroup && adduser -u 1001 -G appgroup -s /bin/sh -D appuser
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/package.json ./
USER appuser
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## Layer Caching

Order from least to most frequently changing:

```dockerfile
# 1. Base image (rarely changes)
FROM node:22-alpine

# 2. System deps (rarely changes)
RUN apk add --no-cache curl

# 3. Package files (changes on dep updates)
COPY package.json bun.lockb ./
RUN npm ci

# 4. Source code (changes frequently)
COPY . .
RUN npm run build
```

## Security

- **Non-root user**: Always `USER appuser` — never run as root
- **Minimal base**: Use `-alpine` or `-slim` variants, or `distroless`
- **No secrets in image**: Use build args or runtime env vars, never `COPY .env`
- **Pin versions**: `FROM node:22.12-alpine` not `FROM node:latest`
- **Scan images**: `trivy image myapp:latest`
- **Read-only filesystem**: Use `--read-only` in compose/k8s where possible

```dockerfile
# .dockerignore (ALWAYS have this)
node_modules
.git
.env*
*.md
dist
coverage
.next
```

## Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/healthz || exit 1
```

## Compose Patterns

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    ports:
      - "3000:3000"
    env_file: .env.local
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

## Build Commands

```bash
docker build -t app:latest .                    # Build
docker build --target builder -t app:test .     # Specific stage
docker compose up -d                            # Start services
docker compose logs -f app                      # Follow logs
docker scout cves app:latest                    # Vulnerability scan
```
