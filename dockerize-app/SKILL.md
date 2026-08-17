---
name: dockerize-app
description: Write clean, small, secure Docker images and Compose setups using multi-stage builds, minimal base images, non-root users, layer caching, .dockerignore, and healthchecks; use when containerizing an application, shrinking or hardening an existing Dockerfile, setting up local multi-service development with docker compose, or when users mention Dockerfile, image size, containers, or compose.
---

# Dockerize App

This skill provides a proven approach to containerizing applications with images that are small, cache-efficient, reproducible, and secure by default. It also covers a clean Docker Compose setup for local multi-service development.

## When to use this skill

- Containerizing a new service for the first time.
- An existing image is too large, slow to build, or runs as root.
- Setting up local development with multiple services (app + database + cache).
- Users mention Dockerfile, image size, layers, non-root, or docker compose.

## Instructions

1. **Pick a minimal, pinned base image.** Prefer slim/alpine/distroless variants and pin a specific version/digest (e.g., `node:20.11-slim`) rather than `latest` for reproducibility.
2. **Use multi-stage builds.** Compile/build in a `builder` stage with full toolchain, then copy only the runtime artifacts into a lean final stage. This keeps compilers and dev dependencies out of the shipped image.
3. **Order layers for caching.** Copy dependency manifests (`package.json`, `go.mod`, `requirements.txt`) and install deps before copying source, so code changes do not bust the dependency cache.
4. **Add a .dockerignore.** Exclude `.git`, `node_modules`, build output, secrets, and local env files to shrink the build context and avoid leaking data.
5. **Run as a non-root user.** Create a dedicated user/group and switch with `USER`. Set an explicit `WORKDIR` and avoid writing to root-owned paths.
6. **Install only production dependencies.** Use `--production`/`--no-dev` flags and clean package caches in the same `RUN` layer to avoid bloat.
7. **Set metadata and runtime config.** Declare `EXPOSE`, sensible `ENV` defaults, and use exec-form `CMD`/`ENTRYPOINT` (`["node","server.js"]`) so signals propagate for graceful shutdown.
8. **Add a healthcheck.** Provide `HEALTHCHECK` so orchestrators know when the container is ready and healthy.
9. **Write a Compose file for local dev.** Define services, networks, named volumes, `depends_on` with health conditions, and env via `env_file`. Never commit real secrets.
10. **Verify.** Build, check final image size, scan for vulnerabilities, and run the container as a smoke test.

## Examples

Multi-stage Node Dockerfile (small, non-root, cache-friendly):

```dockerfile
# ---- builder ----
FROM node:20.11-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---- runtime ----
FROM node:20.11-slim AS runtime
ENV NODE_ENV=production
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev && npm cache clean --force
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "fetch('http://localhost:3000/health').then(r=>process.exit(r.ok?0:1)).catch(()=>process.exit(1))"
CMD ["node", "dist/server.js"]
```

Minimal `.dockerignore`:

```
.git
node_modules
dist
*.log
.env
.env.*
```

Compose with a healthy dependency:

```yaml
services:
  app:
    build: .
    ports: ["3000:3000"]
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes: ["pgdata:/var/lib/postgresql/data"]
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  pgdata:
```

## Checklist

- [ ] Base image is minimal and version-pinned.
- [ ] Multi-stage build keeps build tools out of the final image.
- [ ] Layers ordered so dependency cache survives code changes.
- [ ] `.dockerignore` excludes VCS, deps, build output, and secrets.
- [ ] Container runs as a non-root user.
- [ ] Only production dependencies installed; caches cleaned.
- [ ] Exec-form CMD/ENTRYPOINT and a HEALTHCHECK set.
- [ ] Compose uses env_file, named volumes, and health-gated depends_on.
- [ ] Image built, size checked, and vulnerability-scanned.
