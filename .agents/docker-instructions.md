# Docker Setup Guidelines

## Overview

This guide defines how to containerize applications in the `fuse` monorepo, supporting both **fast local development** with hot-reload and **production-ready deployments**. The setup uses Docker Compose for orchestration and supports multiple applications sharing common services.

## Philosophy

- **Two modes**: Development (hot-reload, volume mounts) and Production (optimized, immutable)
- **Dev/prod parity**: Same database (PostgreSQL) in both environments
- **Simple orchestration**: Docker Compose for all multi-container scenarios
- **Monorepo-aware**: Each app isolated but sharing infrastructure services
- **Security by default**: Non-root users, minimal images, no secrets in code

## When to Use Docker

### Development Mode (Option A - Fast Iteration)
- **Daily development** with hot-reload and instant feedback
- Run entire stack locally: `docker compose up`
- Changes to source code instantly reflected in running containers
- Full dev/prod parity (same database, same environment)

### Production Validation (Option B - Before Release)
- **CI/CD pipelines** and automated testing
- **Pre-release validation** to test production builds locally
- Ensures production image works correctly before deployment
- Use: `docker compose -f docker-compose.prod.yml up`

### Production Deployment
- Cloud platforms (AWS ECS, GCP Cloud Run, Azure Container Apps, etc.)
- Kubernetes clusters
- Any Docker-compatible hosting environment

## Project Structure

```
fuse/
├── docker-compose.yml              # Development: hot-reload, shared services
├── docker-compose.prod.yml         # Production: validation and deployment
├── .env.docker.example             # Template for shared Docker env vars
├── .env.docker                     # Shared config (DB, etc.) - gitignored
├── apps/
│   └── {app-name}/
│       ├── Dockerfile              # Multi-stage: dev + production
│       ├── .dockerignore           # Exclude unnecessary files
│       ├── .env.docker.example     # Template for app-specific vars
│       ├── .env.docker             # App-specific config - gitignored
│       └── src/                    # Source code (mounted in dev mode)
└── .claude/
    └── dockerize-instructions.md   # This file
```

## Quick Start

### Initial Setup

1. **Copy environment templates**:
   ```bash
   # Root-level (shared services)
   cp .env.docker.example .env.docker

   # Per-app (API keys, etc.)
   cp apps/fish/.env.docker.example apps/fish/.env.docker
   ```

2. **Edit environment files** with your configuration:
   ```bash
   # Edit shared config
   nano .env.docker

   # Edit app-specific config
   nano apps/fish/.env.docker
   ```

3. **Start development environment**:
   ```bash
   docker compose up
   ```

### Daily Development

```bash
# Start all services with hot-reload
docker compose up

# Start specific app only
docker compose up fish-dev db

# Rebuild after dependency changes
docker compose up --build

# View logs
docker compose logs -f fish-dev

# Stop everything
docker compose down
```

### Production Validation

```bash
# Test production build locally
docker compose -f docker-compose.prod.yml up --build

# Or test specific app
docker compose -f docker-compose.prod.yml up fish db
```

## Dockerfile Template

Each app should have a multi-stage Dockerfile supporting both development and production.

### Complete Dockerfile: `apps/{app-name}/Dockerfile`

```dockerfile
# =============================================================================
# Stage: base - Shared base configuration
# =============================================================================
FROM node:25-slim AS base

# Install pnpm
RUN corepack enable && corepack prepare pnpm@10 --activate

# Set working directory
WORKDIR /app

# =============================================================================
# Stage: dependencies - Install all dependencies
# =============================================================================
FROM base AS dependencies

# Copy workspace configuration from monorepo root
COPY pnpm-workspace.yaml pnpm-lock.yaml package.json ./

# Copy package.json for this specific app
COPY apps/{app-name}/package.json ./apps/{app-name}/

# Install ALL dependencies (including devDependencies for development)
RUN pnpm install --frozen-lockfile --filter={app-name}

# =============================================================================
# Stage: development - Hot-reload development environment
# =============================================================================
FROM dependencies AS development

# Copy source code
COPY apps/{app-name}/src ./apps/{app-name}/src
COPY apps/{app-name}/tsconfig.json ./apps/{app-name}/
COPY apps/{app-name}/vitest.config.ts ./apps/{app-name}/ 2>/dev/null || true

# Expose port if app uses HTTP (uncomment and adjust)
# EXPOSE 3000

# Development uses tsx watch mode (run via docker-compose command)
# Default command can be overridden in docker-compose.yml
CMD ["pnpm", "--filter", "{app-name}", "dev"]

# =============================================================================
# Stage: build - Build production bundle
# =============================================================================
FROM dependencies AS build

# Copy source code and build configs
COPY apps/{app-name}/src ./apps/{app-name}/src
COPY apps/{app-name}/tsconfig.json ./apps/{app-name}/
COPY apps/{app-name}/esbuild.config.js ./apps/{app-name}/

# Build the application
RUN pnpm --filter={app-name} build

# =============================================================================
# Stage: production - Minimal production runtime
# =============================================================================
FROM node:25-slim AS production

# Install pnpm
RUN corepack enable && corepack prepare pnpm@10 --activate

# Create non-root user for security
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy workspace configuration
COPY --chown=appuser:appuser pnpm-workspace.yaml package.json ./

# Copy app's package.json
COPY --chown=appuser:appuser apps/{app-name}/package.json ./apps/{app-name}/

# Install production dependencies only
RUN pnpm install --frozen-lockfile --filter={app-name} --prod

# Copy built application from build stage
COPY --chown=appuser:appuser --from=build /app/apps/{app-name}/dist ./apps/{app-name}/dist

# Switch to non-root user
USER appuser

# Set NODE_ENV
ENV NODE_ENV=production

# Expose port if needed (uncomment and adjust)
# EXPOSE 3000

# Health check (uncomment and adjust if app has HTTP endpoint)
# HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
#   CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Start the application
CMD ["node", "apps/{app-name}/dist/index.js"]
```

### Key Points

- **Four stages**: `base`, `dependencies`, `development`, `production`
- **Development target**: Includes devDependencies, runs tsx watch mode
- **Production target**: Minimal, optimized, non-root user
- **Build context**: Always from monorepo root (handles workspace structure)
- **Selective copy**: Only necessary files copied to each stage

## .dockerignore Template

Create `.dockerignore` in each app directory to exclude unnecessary files from Docker context.

### `apps/{app-name}/.dockerignore`

```
# Dependencies (will be installed in container)
node_modules
.pnpm-store

# Build output (will be generated in container)
dist
coverage
*.log

# Development files not needed in container
.env*
!.env.docker.example
*.test.ts
*.test.js
**/__tests__
**/__mocks__

# IDE and OS files
.idea
.vscode
.DS_Store
*.swp
*.swo
Thumbs.db

# Git
.git
.gitignore
.gitattributes

# Documentation (unless needed at runtime)
*.md
!README.md

# CI/CD
.github
.gitlab-ci.yml
Dockerfile*
docker-compose*.yml

# Temporary files
tmp
temp
*.tmp
```

## Docker Compose Configuration

### Development: `docker-compose.yml`

This is the primary development environment with hot-reload support.

```yaml
version: '3.9'

services:
  # =============================================================================
  # Application: fish (development with hot-reload)
  # =============================================================================
  fish-dev:
    build:
      context: .                        # Build from monorepo root
      dockerfile: apps/fish/Dockerfile
      target: development               # Use development stage
    container_name: fuse-fish-dev
    restart: unless-stopped
    volumes:
      # Mount source code for hot-reload
      - ./apps/fish/src:/app/apps/fish/src:cached
      # Mount config files
      - ./apps/fish/tsconfig.json:/app/apps/fish/tsconfig.json:ro
      - ./apps/fish/package.json:/app/apps/fish/package.json:ro
      # Optional: mount test files if running tests in container
      # - ./apps/fish/vitest.config.ts:/app/apps/fish/vitest.config.ts:ro
    environment:
      NODE_ENV: development
      # Database connection (shared service)
      DATABASE_URL: postgresql://fuse:fuse_dev_password@db:5432/fuse
      # Load app-specific vars from file
    env_file:
      - apps/fish/.env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - fish-network
      - shared-services
    # Uncomment if app exposes HTTP port
    # ports:
    #   - "3000:3000"

  # =============================================================================
  # Shared Service: PostgreSQL Database
  # =============================================================================
  db:
    image: postgres:16-alpine
    container_name: fuse-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: fuse                 # Shared database for all apps
      POSTGRES_USER: fuse
      POSTGRES_PASSWORD: fuse_dev_password
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=C"
    volumes:
      # Persist database data
      - postgres-data:/var/lib/postgresql/data
      # Optional: init scripts for schema setup
      # - ./database/init:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"                     # Expose for local tools (psql, GUI clients)
    networks:
      - shared-services
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fuse -d fuse"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  # =============================================================================
  # Future App Example (commented out)
  # =============================================================================
  # future-app-dev:
  #   build:
  #     context: .
  #     dockerfile: apps/future-app/Dockerfile
  #     target: development
  #   container_name: fuse-future-app-dev
  #   volumes:
  #     - ./apps/future-app/src:/app/apps/future-app/src:cached
  #   environment:
  #     DATABASE_URL: postgresql://fuse:fuse_dev_password@db:5432/fuse
  #   env_file:
  #     - apps/future-app/.env.docker
  #   depends_on:
  #     db:
  #       condition: service_healthy
  #   networks:
  #     - future-app-network
  #     - shared-services

# =============================================================================
# Networks: Isolated per-app + shared services
# =============================================================================
networks:
  shared-services:
    driver: bridge
    name: fuse-shared-services

  fish-network:
    driver: bridge
    name: fuse-fish-network

  # future-app-network:
  #   driver: bridge
  #   name: fuse-future-app-network

# =============================================================================
# Volumes: Persistent data storage
# =============================================================================
volumes:
  postgres-data:
    name: fuse-postgres-data
```

### Production: `docker-compose.prod.yml`

Use this for production validation and deployment.

```yaml
version: '3.9'

services:
  # =============================================================================
  # Application: fish (production build)
  # =============================================================================
  fish:
    build:
      context: .
      dockerfile: apps/fish/Dockerfile
      target: production                # Use production stage
    image: fuse-fish:latest
    container_name: fuse-fish
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://fuse:${POSTGRES_PASSWORD}@db:5432/fuse
    env_file:
      - apps/fish/.env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - fish-network
      - shared-services
    # Uncomment for HTTP apps
    # ports:
    #   - "3000:3000"

  # =============================================================================
  # Shared Service: PostgreSQL Database
  # =============================================================================
  db:
    image: postgres:16-alpine
    container_name: fuse-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: fuse
      POSTGRES_USER: fuse
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}  # From .env.docker
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - shared-services
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U fuse -d fuse"]
      interval: 10s
      timeout: 5s
      retries: 5

# =============================================================================
# Networks
# =============================================================================
networks:
  shared-services:
    driver: bridge
  fish-network:
    driver: bridge

# =============================================================================
# Volumes
# =============================================================================
volumes:
  postgres-data:
```

## Environment Variables

### Root-Level: `.env.docker.example`

Template for shared infrastructure configuration.

```bash
# =============================================================================
# Shared Docker Environment Variables
# Copy this file to .env.docker and customize
# =============================================================================

# PostgreSQL Database
POSTGRES_PASSWORD=changeme_in_production

# Add other shared service configs here
# REDIS_PASSWORD=changeme
```

### App-Level: `apps/{app-name}/.env.docker.example`

Template for app-specific configuration.

```bash
# =============================================================================
# {App Name} - Docker Environment Variables
# Copy this file to .env.docker and customize
# =============================================================================

# External API Configuration
OBJEKTBANKEN_API_KEY=your_api_key_here
OBJEKTBANKEN_API_BASEURL=https://api.objektbanken.com

# App-specific settings
# LOG_LEVEL=info
# FEATURE_FLAG_XYZ=true
```

### Using Environment Variables

```bash
# Development
docker compose up

# Production (with overrides)
POSTGRES_PASSWORD=secure_prod_password \
  docker compose -f docker-compose.prod.yml up

# Or use .env file for production secrets (not committed)
echo "POSTGRES_PASSWORD=secure_prod_password" > .env.docker
docker compose -f docker-compose.prod.yml up
```

## Common Commands

### Development Workflow

```bash
# Start all services
docker compose up

# Start in background
docker compose up -d

# Start specific app + dependencies
docker compose up fish-dev db

# Rebuild after package.json changes
docker compose up --build fish-dev

# View logs
docker compose logs -f fish-dev

# Execute commands in running container
docker compose exec fish-dev pnpm --filter fish test
docker compose exec fish-dev sh  # Open shell

# Stop all services
docker compose down

# Stop and remove volumes (fresh start)
docker compose down -v
```

### Production Validation

```bash
# Build and run production images
docker compose -f docker-compose.prod.yml up --build

# Run in background
docker compose -f docker-compose.prod.yml up -d

# Check logs
docker compose -f docker-compose.prod.yml logs -f fish

# Stop
docker compose -f docker-compose.prod.yml down
```

### Database Management

```bash
# Connect to PostgreSQL with psql
docker compose exec db psql -U fuse -d fuse

# Backup database
docker compose exec db pg_dump -U fuse fuse > backup.sql

# Restore database
docker compose exec -T db psql -U fuse -d fuse < backup.sql

# View database logs
docker compose logs db
```

### Cleanup

```bash
# Remove stopped containers
docker compose down

# Remove containers and volumes (data loss!)
docker compose down -v

# Remove all unused Docker resources
docker system prune -a --volumes
```

## Adding a New App to the Monorepo

Follow these steps to dockerize a new application:

### 1. Create Dockerfile

Copy and customize the template:

```bash
# Create Dockerfile for new app
cp apps/fish/Dockerfile apps/new-app/Dockerfile

# Edit and replace {app-name} with 'new-app'
nano apps/new-app/Dockerfile
```

### 2. Create .dockerignore

```bash
cp apps/fish/.dockerignore apps/new-app/.dockerignore
```

### 3. Create Environment Template

```bash
# Create app-specific env template
cat > apps/new-app/.env.docker.example << 'EOF'
# New App - Docker Environment Variables
# Copy to .env.docker and customize

# Add your app-specific variables here
API_KEY=changeme
EOF

# Create actual env file
cp apps/new-app/.env.docker.example apps/new-app/.env.docker

# Add to .gitignore
echo "apps/new-app/.env.docker" >> .gitignore
```

### 4. Add to docker-compose.yml

```yaml
services:
  # ... existing services ...

  new-app-dev:
    build:
      context: .
      dockerfile: apps/new-app/Dockerfile
      target: development
    container_name: fuse-new-app-dev
    restart: unless-stopped
    volumes:
      - ./apps/new-app/src:/app/apps/new-app/src:cached
      - ./apps/new-app/tsconfig.json:/app/apps/new-app/tsconfig.json:ro
      - ./apps/new-app/package.json:/app/apps/new-app/package.json:ro
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://fuse:fuse_dev_password@db:5432/fuse
    env_file:
      - apps/new-app/.env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - new-app-network
      - shared-services
    # ports:
    #   - "3001:3000"  # Adjust port to avoid conflicts

networks:
  # ... existing networks ...
  new-app-network:
    driver: bridge
    name: fuse-new-app-network
```

### 5. Add to docker-compose.prod.yml

```yaml
services:
  new-app:
    build:
      context: .
      dockerfile: apps/new-app/Dockerfile
      target: production
    image: fuse-new-app:latest
    container_name: fuse-new-app
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://fuse:${POSTGRES_PASSWORD}@db:5432/fuse
    env_file:
      - apps/new-app/.env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - new-app-network
      - shared-services

networks:
  new-app-network:
    driver: bridge
```

### 6. Test the Setup

```bash
# Test development build
docker compose up new-app-dev --build

# Test production build
docker compose -f docker-compose.prod.yml up new-app --build

# Run both apps simultaneously
docker compose up fish-dev new-app-dev
```

## Network Architecture

### Isolated Networks with Shared Services

Each app runs in its own isolated network but can access shared services:

```
┌─────────────────────────────────────────────────┐
│              fuse-shared-services               │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │         PostgreSQL Database               │  │
│  │         (fuse-db)                         │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
└────────┬─────────────────────────────┬──────────┘
         │                             │
         │                             │
┌────────┴─────────┐         ┌────────┴─────────┐
│  fuse-fish-network│         │ fuse-app2-network│
│                   │         │                   │
│  ┌─────────────┐ │         │  ┌─────────────┐ │
│  │  fish-dev   │ │         │  │  app2-dev   │ │
│  └─────────────┘ │         │  └─────────────┘ │
│                   │         │                   │
└───────────────────┘         └───────────────────┘
```

**Key Points:**
- Apps cannot directly communicate with each other (isolated)
- All apps can access shared database
- Easy to add inter-app communication later by connecting networks
- Security: minimal network exposure

### Connecting Apps (If Needed Later)

To allow two apps to communicate:

```yaml
services:
  fish-dev:
    networks:
      - fish-network
      - shared-services
      - inter-app-communication  # Add this

  new-app-dev:
    networks:
      - new-app-network
      - shared-services
      - inter-app-communication  # Add this

networks:
  inter-app-communication:
    driver: bridge
```

## Volume Mounts for Development

### Types of Mounts

1. **Source code (cached)**: Hot-reload, instant feedback
   ```yaml
   volumes:
     - ./apps/fish/src:/app/apps/fish/src:cached
   ```

2. **Config files (read-only)**: Prevent accidental changes
   ```yaml
   volumes:
     - ./apps/fish/tsconfig.json:/app/apps/fish/tsconfig.json:ro
   ```

3. **Node modules (not mounted)**: Keep container's installation
   - Don't mount `node_modules` - let container manage it
   - If needed, use named volume to cache

### Performance Optimization

On macOS and Windows, use `:cached` for better performance:

```yaml
volumes:
  # Good: cached delegated mount
  - ./apps/fish/src:/app/apps/fish/src:cached

  # Avoid: not cached (slow on macOS/Windows)
  - ./apps/fish/src:/app/apps/fish/src
```

## Troubleshooting

### Build Fails with "Cannot find module"

**Problem**: Dependencies not installed or path aliases not resolved.

**Solution**:
```bash
# Rebuild without cache
docker compose build --no-cache fish-dev

# Check if package.json changed
docker compose up --build fish-dev

# Verify path aliases in tsconfig.json
docker compose exec fish-dev cat apps/fish/tsconfig.json
```

### Hot Reload Not Working

**Problem**: Changes to source files don't reflect in running container.

**Solution**:
```bash
# Check volume mounts
docker compose exec fish-dev ls -la /app/apps/fish/src

# Verify tsx is running in watch mode
docker compose logs fish-dev | grep tsx

# Restart the service
docker compose restart fish-dev
```

### Database Connection Refused

**Problem**: App can't connect to PostgreSQL.

**Solution**:
```bash
# Check database is healthy
docker compose ps db

# Check database logs
docker compose logs db

# Verify connection string
docker compose exec fish-dev env | grep DATABASE_URL

# Manually test connection
docker compose exec fish-dev ping db
docker compose exec fish-dev nc -zv db 5432
```

### Container Exits Immediately

**Problem**: Container starts then stops right away.

**Solution**:
```bash
# Check logs for errors
docker compose logs fish-dev

# Run with interactive shell to debug
docker compose run --rm fish-dev sh

# Check if environment variables are set
docker compose exec fish-dev env
```

### Port Already in Use

**Problem**: `Error: bind: address already in use`

**Solution**:
```bash
# Find process using the port
lsof -i :5432
# or
netstat -an | grep 5432

# Stop conflicting service
brew services stop postgresql  # macOS
sudo systemctl stop postgresql # Linux

# Or change port in docker-compose.yml
ports:
  - "5433:5432"  # Use different host port
```

### Permission Errors in Production Image

**Problem**: Container can't write files or access resources.

**Solution**:
```dockerfile
# Ensure proper ownership
COPY --chown=appuser:appuser --from=build /app/apps/fish/dist ./apps/fish/dist

# Create necessary directories with correct permissions
RUN mkdir -p /app/data && chown appuser:appuser /app/data
```

### Volume Data Persists After `down`

**Problem**: Old data remains after stopping containers.

**Solution**:
```bash
# Remove volumes when stopping
docker compose down -v

# Or remove specific volume
docker volume rm fuse-postgres-data

# List all volumes
docker volume ls
```

## Security Best Practices

### 1. Non-Root User (Production Only)

Development runs as root for convenience, production uses non-root:

```dockerfile
# Production stage
USER appuser
```

### 2. Environment Variables

**Never commit secrets:**
```bash
# ❌ Bad - committed to git
DATABASE_URL=postgresql://user:password@localhost/db

# ✅ Good - in .env.docker (gitignored)
echo "DATABASE_URL=postgresql://user:password@localhost/db" > .env.docker
```

**Use separate files:**
- `.env.docker.example` - Template, committed
- `.env.docker` - Actual secrets, gitignored

### 3. Network Isolation

Apps are isolated by default:
```yaml
networks:
  - app-specific-network  # Isolated
  - shared-services       # Only for DB/Redis/etc.
```

### 4. Read-Only Mounts

Use `:ro` for files that shouldn't change:
```yaml
volumes:
  - ./apps/fish/package.json:/app/apps/fish/package.json:ro
```

### 5. Minimal Base Images

Use slim variants:
```dockerfile
FROM node:25-slim  # Not node:25 (much larger)
```

### 6. Explicit Dependencies

Pin exact versions:
```dockerfile
FROM node:25-slim              # ❌ Can change
FROM node:25.0.0-slim          # ✅ Reproducible
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Build and Test with Docker

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up environment files
        run: |
          cp .env.docker.example .env.docker
          cp apps/fish/.env.docker.example apps/fish/.env.docker
          # Add secrets from GitHub Secrets
          echo "OBJEKTBANKEN_API_KEY=${{ secrets.OBJEKTBANKEN_API_KEY }}" >> apps/fish/.env.docker

      - name: Build development images
        run: docker compose build fish-dev

      - name: Run tests in container
        run: docker compose run --rm fish-dev pnpm --filter fish test

      - name: Build production image
        run: docker compose -f docker-compose.prod.yml build fish

      - name: Test production image
        run: docker compose -f docker-compose.prod.yml up -d fish

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Build and push to registry
        run: |
          docker compose -f docker-compose.prod.yml build fish
          # Add steps to tag and push to your registry
```

## Checklist: Dockerizing a New App

Use this checklist when adding a new app to the monorepo:

- [ ] Create `Dockerfile` from template
- [ ] Create `.dockerignore`
- [ ] Replace all `{app-name}` placeholders with actual app name
- [ ] Create `.env.docker.example` with required variables
- [ ] Create `.env.docker` and add to `.gitignore`
- [ ] Add service to `docker-compose.yml` (development)
- [ ] Add service to `docker-compose.prod.yml` (production)
- [ ] Create isolated network for the app
- [ ] Adjust ports if app exposes HTTP (avoid conflicts)
- [ ] Test development build: `docker compose up {app}-dev --build`
- [ ] Verify hot-reload works: change source file, check logs
- [ ] Test production build: `docker compose -f docker-compose.prod.yml up {app} --build`
- [ ] Test database connectivity from app
- [ ] Verify non-root user in production: `docker compose -f docker-compose.prod.yml exec {app} id`
- [ ] Check final image size: `docker images fuse-{app}`
- [ ] Document app-specific Docker notes in app's README (if any)
- [ ] Update CI/CD workflows if needed

## Performance Tips

### 1. Layer Caching

Order Dockerfile instructions from least to most frequently changed:

```dockerfile
# ✅ Good: stable layers first
COPY pnpm-lock.yaml package.json ./
RUN pnpm install
COPY src ./src
RUN pnpm build

# ❌ Bad: invalidates cache on every code change
COPY . .
RUN pnpm install && pnpm build
```

### 2. BuildKit Cache Mounts

Enable BuildKit for faster builds:

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Or in docker-compose.yml
COMPOSE_DOCKER_CLI_BUILD=1 DOCKER_BUILDKIT=1 docker compose build
```

### 3. Prune Regularly

Clean up unused resources:

```bash
# Remove unused images, containers, networks
docker system prune -a

# Remove unused volumes (careful: data loss!)
docker volume prune
```

### 4. Multi-Stage Builds

Always use multi-stage builds to minimize final image size:

```dockerfile
# Build stage: large (includes build tools)
FROM node:25-slim AS build
# ... build steps ...

# Production stage: small (only runtime)
FROM node:25-slim AS production
COPY --from=build /app/dist ./dist
```

## Additional Resources

- [Docker Compose Specification](https://docs.docker.com/compose/compose-file/)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [pnpm Docker Guide](https://pnpm.io/docker)
- [Node.js Docker Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres)

## Notes for AI/Claude Code

When dockerizing an application in this monorepo:

1. **Always use the templates** - Consistency is critical
2. **Replace `{app-name}` everywhere** - Dockerfile, docker-compose.yml, networks, volumes
3. **Test both modes** - Development (hot-reload) and Production (validation)
4. **Verify hot-reload** - Make a code change and check it reflects immediately
5. **Check networks** - Each app should be isolated with own network
6. **Validate database access** - Every app should connect to shared DB successfully
7. **Document deviations** - If template doesn't work, document why and update this guide
8. **Keep it simple** - Don't add complexity unless required
9. **Security first** - Non-root in production, secrets in env files, minimal images

Remember: **Two modes** - Development (fast iteration) and Production (validation/deployment). Both must work correctly.
