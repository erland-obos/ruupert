# Docker Setup Guidelines

## Overview

This guide defines how to containerize applications, supporting both **fast local development** with hot-reload and *
*production-ready deployments**. The setup uses Docker Compose for orchestration and can be adapted for both single
applications and monorepo structures.

## Philosophy

- **Two modes**: Development (hot-reload, volume mounts) and Production (optimized, immutable)
- **Dev/prod parity**: Same database type in both environments
- **Simple orchestration**: Docker Compose for all multi-container scenarios
- **Security by default**: Non-root users, minimal images, no secrets in code
- **Flexible architecture**: Supports both single-app and multi-app setups

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

## Directory Structure

### Single Application

```
project/
├── Dockerfile                      # Multi-stage: dev + production
├── docker-compose.yml              # Development: hot-reload
├── docker-compose.prod.yml         # Production: validation and deployment
├── .dockerignore                   # Exclude unnecessary files
├── .env.docker.example             # Template for Docker env vars
├── .env.docker                     # Actual config - gitignored
└── src/                            # Source code (mounted in dev mode)
```

### Monorepo Structure

```
project/
├── docker-compose.yml              # Development: hot-reload, shared services
├── docker-compose.prod.yml         # Production: validation and deployment
├── .env.docker.example             # Template for shared Docker env vars
├── .env.docker                     # Shared config (DB, etc.) - gitignored
└── apps/
    └── {app-name}/
        ├── Dockerfile              # Multi-stage: dev + production
        ├── .dockerignore           # Exclude unnecessary files
        ├── .env.docker.example     # Template for app-specific vars
        ├── .env.docker             # App-specific config - gitignored
        └── src/                    # Source code (mounted in dev mode)
```

## Quick Start

### Initial Setup

1. **Copy environment templates**:
   ```bash
   # For single app
   cp .env.docker.example .env.docker

   # For monorepo (shared services)
   cp .env.docker.example .env.docker

   # For monorepo (per-app)
   cp apps/{app-name}/.env.docker.example apps/{app-name}/.env.docker
   ```

2. **Edit environment files** with your configuration:
   ```bash
   # Edit shared or app config
   nano .env.docker
   ```

3. **Start development environment**:
   ```bash
   docker compose up
   ```

### Daily Development

```bash
# Start all services with hot-reload
docker compose up

# Start specific app only (monorepo)
docker compose up {app-name}-dev db

# Rebuild after dependency changes
docker compose up --build

# View logs
docker compose logs -f {app-name}-dev

# Stop everything
docker compose down
```

### Production Validation

```bash
# Test production build locally
docker compose -f docker-compose.prod.yml up --build

# Or test specific app
docker compose -f docker-compose.prod.yml up {app-name}
```

## Dockerfile Template

Each app should have a multi-stage Dockerfile supporting both development and production.

### Single Application: `Dockerfile`

```dockerfile
# =============================================================================
# Stage: base - Shared base configuration
# =============================================================================
FROM node:25-slim AS base

# Install package manager (pnpm example, adjust for npm/yarn)
RUN corepack enable && corepack prepare pnpm@10 --activate

# Set working directory
WORKDIR /app

# =============================================================================
# Stage: dependencies - Install all dependencies
# =============================================================================
FROM base AS dependencies

# Copy dependency manifests
COPY package.json pnpm-lock.yaml ./

# Install ALL dependencies (including devDependencies for development)
RUN pnpm install --frozen-lockfile

# =============================================================================
# Stage: development - Hot-reload development environment
# =============================================================================
FROM dependencies AS development

# Copy source code
COPY src ./src
COPY tsconfig.json ./

# Expose port if app uses HTTP (uncomment and adjust)
# EXPOSE 3000

# Development uses watch mode (e.g., tsx, nodemon)
CMD ["pnpm", "dev"]

# =============================================================================
# Stage: build - Build production bundle
# =============================================================================
FROM dependencies AS build

# Copy source code and build configs
COPY src ./src
COPY tsconfig.json ./
COPY esbuild.config.js ./

# Build the application
RUN pnpm build

# =============================================================================
# Stage: production - Minimal production runtime
# =============================================================================
FROM node:25-slim AS production

# Install package manager
RUN corepack enable && corepack prepare pnpm@10 --activate

# Create non-root user for security
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy dependency manifests
COPY --chown=appuser:appuser package.json pnpm-lock.yaml ./

# Install production dependencies only
RUN pnpm install --frozen-lockfile --prod

# Copy built application from build stage
COPY --chown=appuser:appuser --from=build /app/dist ./dist

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
CMD ["node", "dist/index.js"]
```

### Monorepo Application: `apps/{app-name}/Dockerfile`

```dockerfile
# =============================================================================
# Stage: base - Shared base configuration
# =============================================================================
FROM node:25-slim AS base

# Install pnpm (or your workspace package manager)
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

# Expose port if app uses HTTP (uncomment and adjust)
# EXPOSE 3000

# Development uses workspace filter command
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
- **Development target**: Includes devDependencies, runs watch mode
- **Production target**: Minimal, optimized, non-root user
- **Build context**: For monorepo, always from root (handles workspace structure)
- **Selective copy**: Only necessary files copied to each stage

## .dockerignore Template

Create `.dockerignore` in each app directory (or project root for single apps) to exclude unnecessary files from Docker
context.

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

#### Single Application

```yaml
version: '3.9'

services:
  app-dev:
    build:
      context: ../..
      dockerfile: Dockerfile
      target: development
    container_name: { project-name }-app-dev
    restart: unless-stopped
    volumes:
      # Mount source code for hot-reload
      - ./src:/app/src:cached
      # Mount config files (read-only)
      - ./tsconfig.json:/app/tsconfig.json:ro
      - ./package.json:/app/package.json:ro
    environment:
      NODE_ENV: development
      # Add your environment variables here
      DATABASE_URL: postgresql://user:password@db:5432/dbname
    env_file:
      - .env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    # Uncomment if app exposes HTTP port
    # ports:
    #   - "3000:3000"

  # Example: PostgreSQL database (adjust for your needs)
  db:
    image: postgres:16-alpine
    container_name: { project-name }-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: { database-name }
      POSTGRES_USER: { database-user }
      POSTGRES_PASSWORD: { database-password }
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U {database-user} -d {database-name}" ]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
```

#### Monorepo with Multiple Applications

```yaml
version: '3.9'

services:
  # =============================================================================
  # Application 1 (development with hot-reload)
  # =============================================================================
  { app-name }-dev:
                build:
                  context: .                        # Build from monorepo root
                  dockerfile: apps/{app-name}/Dockerfile
                  target: development
                container_name: { project-name }-{app-name}-dev
                restart: unless-stopped
                volumes:
                  # Mount source code for hot-reload
                  - ./apps/{app-name}/src:/app/apps/{app-name}/src:cached
                  # Mount config files (read-only)
                  - ./apps/{app-name}/tsconfig.json:/app/apps/{app-name}/tsconfig.json:ro
                  - ./apps/{app-name}/package.json:/app/apps/{app-name}/package.json:ro
                environment:
                  NODE_ENV: development
                  DATABASE_URL: postgresql://user:password@db:5432/dbname
                env_file:
                  - apps/{app-name}/.env.docker
                depends_on:
                  db:
                    condition: service_healthy
                networks:
                  - { app-name }-network
                  - shared-services
                # Uncomment if app exposes HTTP port
                # ports:
                #   - "3000:3000"

  # =============================================================================
  # Shared Service: Database
  # =============================================================================
              db:
                image: postgres:16-alpine
                container_name: { project-name }-db
                restart: unless-stopped
                environment:
                  POSTGRES_DB: { database-name }
                  POSTGRES_USER: { database-user }
                  POSTGRES_PASSWORD: { database-password }
                volumes:
                  - postgres-data:/var/lib/postgresql/data
                ports:
                  - "5432:5432"
                networks:
                  - shared-services
                healthcheck:
                  test: [ "CMD-SHELL", "pg_isready -U {database-user} -d {database-name}" ]
                  interval: 10s
                  timeout: 5s
                  retries: 5
                  start_period: 10s

# =============================================================================
# Networks: Isolated per-app + shared services
# =============================================================================
networks:
              shared-services:
                driver: bridge

  { app-name }-network:
                driver: bridge

# =============================================================================
# Volumes: Persistent data storage
# =============================================================================
volumes:
  postgres-data:
```

### Production: `docker-compose.prod.yml`

#### Single Application

```yaml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    image: { project-name }-app:latest
    container_name: { project-name }-app
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://user:${DB_PASSWORD}@db:5432/dbname
    env_file:
      - .env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    # Uncomment for HTTP apps
    # ports:
    #   - "3000:3000"

  db:
    image: postgres:16-alpine
    container_name: { project-name }-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: { database-name }
      POSTGRES_USER: { database-user }
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U {database-user} -d {database-name}" ]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
```

#### Monorepo with Multiple Applications

```yaml
version: '3.9'

services:
  { app-name }:
    build:
      context: .
      dockerfile: apps/{app-name}/Dockerfile
      target: production
    image: { project-name }-{app-name}:latest
    container_name: { project-name }-{app-name}
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://user:${DB_PASSWORD}@db:5432/dbname
    env_file:
      - apps/{app-name}/.env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - { app-name }-network
      - shared-services
    # Uncomment for HTTP apps
    # ports:
    #   - "3000:3000"

    db:
      image: postgres:16-alpine
      container_name: { project-name }-db
      restart: unless-stopped
      environment:
        POSTGRES_DB: { database-name }
        POSTGRES_USER: { database-user }
        POSTGRES_PASSWORD: ${DB_PASSWORD}
      volumes:
        - postgres-data:/var/lib/postgresql/data
      networks:
        - shared-services
      healthcheck:
        test: [ "CMD-SHELL", "pg_isready -U {database-user} -d {database-name}" ]
        interval: 10s
        timeout: 5s
        retries: 5

networks:
              shared-services:
                driver: bridge
  { app-name }-network:
                driver: bridge

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

# Database
DB_PASSWORD=changeme_in_production

# Add other shared service configs here
# REDIS_PASSWORD=changeme
```

### App-Level: `apps/{app-name}/.env.docker.example` (Monorepo)

Template for app-specific configuration.

```bash
# =============================================================================
# {App Name} - Docker Environment Variables
# Copy this file to .env.docker and customize
# =============================================================================

# External API Configuration
EXTERNAL_API_KEY=your_api_key_here
EXTERNAL_API_URL=https://api.example.com

# App-specific settings
LOG_LEVEL=info
# FEATURE_FLAG_XYZ=true
```

### Single App: `.env.docker.example`

```bash
# =============================================================================
# Application - Docker Environment Variables
# Copy this file to .env.docker and customize
# =============================================================================

# Database
DB_PASSWORD=changeme_in_production

# External API Configuration
EXTERNAL_API_KEY=your_api_key_here
EXTERNAL_API_URL=https://api.example.com

# App-specific settings
LOG_LEVEL=info
```

### Using Environment Variables

```bash
# Development
docker compose up

# Production (with overrides)
DB_PASSWORD=secure_prod_password \
  docker compose -f docker-compose.prod.yml up

# Or use .env file for production secrets (not committed)
echo "DB_PASSWORD=secure_prod_password" > .env.docker
docker compose -f docker-compose.prod.yml up
```

## Common Commands

### Development Workflow

```bash
# Start all services
docker compose up

# Start in background
docker compose up -d

# Start specific app + dependencies (monorepo)
docker compose up {app-name}-dev db

# Rebuild after package.json changes
docker compose up --build {app-name}-dev

# View logs
docker compose logs -f {app-name}-dev

# Execute commands in running container
docker compose exec {app-name}-dev pnpm test
docker compose exec {app-name}-dev sh  # Open shell

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
docker compose -f docker-compose.prod.yml logs -f {app-name}

# Stop
docker compose -f docker-compose.prod.yml down
```

### Database Management (PostgreSQL Example)

```bash
# Connect to PostgreSQL with psql
docker compose exec db psql -U {database-user} -d {database-name}

# Backup database
docker compose exec db pg_dump -U {database-user} {database-name} > backup.sql

# Restore database
docker compose exec -T db psql -U {database-user} -d {database-name} < backup.sql

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

## Network Architecture

### Simple Single-App Network

```
┌─────────────────────────┐
│     app-network         │
│                         │
│  ┌────────┐  ┌────────┐│
│  │  app   │  │   db   ││
│  └────────┘  └────────┘│
│                         │
└─────────────────────────┘
```

### Monorepo: Isolated Networks with Shared Services

Each app runs in its own isolated network but can access shared services:

```
┌─────────────────────────────────────────────────┐
│              shared-services                    │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │         Database / Redis / etc.          │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
└────────┬─────────────────────────────┬──────────┘
         │                             │
         │                             │
┌────────┴─────────┐         ┌────────┴─────────┐
│  app1-network    │         │  app2-network    │
│                  │         │                  │
│  ┌────────────┐  │         │  ┌────────────┐  │
│  │   app1     │  │         │  │   app2     │  │
│  └────────────┘  │         │  └────────────┘  │
│                  │         │                  │
└──────────────────┘         └──────────────────┘
```

**Key Points:**

- Apps cannot directly communicate with each other (isolated)
- All apps can access shared services (database, cache, etc.)
- Easy to add inter-app communication later by connecting networks
- Security: minimal network exposure

### Connecting Apps (If Needed Later)

To allow two apps to communicate:

```yaml
services:
  app1-dev:
    networks:
      - app1-network
      - shared-services
      - inter-app-communication  # Add this

  app2-dev:
    networks:
      - app2-network
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
     - ./src:/app/src:cached
   ```

2. **Config files (read-only)**: Prevent accidental changes
   ```yaml
   volumes:
     - ./tsconfig.json:/app/tsconfig.json:ro
   ```

3. **Node modules (not mounted)**: Keep container's installation
    - Don't mount `node_modules` - let container manage it
    - If needed, use named volume to cache

### Performance Optimization

On macOS and Windows, use `:cached` for better performance:

```yaml
volumes:
  # Good: cached delegated mount
  - ./src:/app/src:cached

  # Avoid: not cached (slow on macOS/Windows)
  - ./src:/app/src
```

## Troubleshooting

### Build Fails with "Cannot find module"

**Problem**: Dependencies not installed or path aliases not resolved.

**Solution**:

```bash
# Rebuild without cache
docker compose build --no-cache {app-name}-dev

# Check if package.json changed
docker compose up --build {app-name}-dev

# Verify path aliases in tsconfig.json
docker compose exec {app-name}-dev cat tsconfig.json
```

### Hot Reload Not Working

**Problem**: Changes to source files don't reflect in running container.

**Solution**:

```bash
# Check volume mounts
docker compose exec {app-name}-dev ls -la /app/src

# Verify watch mode is running
docker compose logs {app-name}-dev

# Restart the service
docker compose restart {app-name}-dev
```

### Database Connection Refused

**Problem**: App can't connect to database.

**Solution**:

```bash
# Check database is healthy
docker compose ps db

# Check database logs
docker compose logs db

# Verify connection string
docker compose exec {app-name}-dev env | grep DATABASE_URL

# Manually test connection
docker compose exec {app-name}-dev ping db
docker compose exec {app-name}-dev nc -zv db 5432
```

### Container Exits Immediately

**Problem**: Container starts then stops right away.

**Solution**:

```bash
# Check logs for errors
docker compose logs {app-name}-dev

# Run with interactive shell to debug
docker compose run --rm {app-name}-dev sh

# Check if environment variables are set
docker compose exec {app-name}-dev env
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
COPY --chown=appuser:appuser --from=build /app/dist ./dist

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
docker volume rm {project-name}-postgres-data

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
  - ./package.json:/app/package.json:ro
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
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up environment files
        run: |
          cp .env.docker.example .env.docker
          # Add secrets from GitHub Secrets
          echo "EXTERNAL_API_KEY=${{ secrets.EXTERNAL_API_KEY }}" >> .env.docker

      - name: Build development images
        run: docker compose build

      - name: Run tests in container
        run: docker compose run --rm app-dev pnpm test

      - name: Build production image
        run: docker compose -f docker-compose.prod.yml build

      - name: Test production image
        run: docker compose -f docker-compose.prod.yml up -d

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Build and push to registry
        run: |
          docker compose -f docker-compose.prod.yml build
          # Add steps to tag and push to your registry
```

## Database Options

While examples in this guide use PostgreSQL, Docker supports any database:

### PostgreSQL

```yaml
db:
  image: postgres:16-alpine
  environment:
    POSTGRES_DB: { database-name }
    POSTGRES_USER: { database-user }
    POSTGRES_PASSWORD: ${DB_PASSWORD}
  healthcheck:
    test: [ "CMD-SHELL", "pg_isready -U {database-user}" ]
```

### MySQL

```yaml
db:
  image: mysql:8.0
  environment:
    MYSQL_DATABASE: { database-name }
    MYSQL_USER: { database-user }
    MYSQL_PASSWORD: ${DB_PASSWORD}
    MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
  healthcheck:
    test: [ "CMD", "mysqladmin", "ping", "-h", "localhost" ]
```

### MongoDB

```yaml
db:
  image: mongo:7
  environment:
    MONGO_INITDB_ROOT_USERNAME: { database-user }
    MONGO_INITDB_ROOT_PASSWORD: ${DB_PASSWORD}
  healthcheck:
    test: [ "CMD", "mongosh", "--eval", "db.adminCommand('ping')" ]
```

### Redis

```yaml
cache:
  image: redis:7-alpine
  environment:
    REDIS_PASSWORD: ${REDIS_PASSWORD}
  command: redis-server --requirepass ${REDIS_PASSWORD}
  healthcheck:
    test: [ "CMD", "redis-cli", "ping" ]
```

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

## Checklist: Dockerizing an Application

Use this checklist when setting up Docker:

- [ ] Create `Dockerfile` from template (single-app or monorepo)
- [ ] Create `.dockerignore`
- [ ] Replace all `{placeholders}` with actual names
- [ ] Create `.env.docker.example` with required variables
- [ ] Create `.env.docker` and add to `.gitignore`
- [ ] Create `docker-compose.yml` (development)
- [ ] Create `docker-compose.prod.yml` (production)
- [ ] Configure networks (isolated if monorepo, simple if single-app)
- [ ] Adjust ports if app exposes HTTP (avoid conflicts)
- [ ] Test development build: `docker compose up --build`
- [ ] Verify hot-reload works: change source file, check logs
- [ ] Test production build: `docker compose -f docker-compose.prod.yml up --build`
- [ ] Test database connectivity from app
- [ ] Verify non-root user in production: `docker compose -f docker-compose.prod.yml exec {app} id`
- [ ] Check final image size: `docker images`
- [ ] Document app-specific Docker notes if needed
- [ ] Update CI/CD workflows if applicable

## Additional Resources

- [Docker Compose Specification](https://docs.docker.com/compose/compose-file/)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Node.js Docker Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)

## Notes for AI Assistants

When dockerizing an application:

1. **Use the templates** - Consistency is critical
2. **Replace all placeholders** - `{project-name}`, `{app-name}`, `{database-name}`, etc.
3. **Test both modes** - Development (hot-reload) and Production (validation)
4. **Verify hot-reload** - Make a code change and check it reflects immediately
5. **Configure networks appropriately** - Isolated for monorepo, simple for single-app
6. **Validate database access** - Ensure connectivity works
7. **Document deviations** - If template doesn't work, document why
8. **Keep it simple** - Don't add complexity unless required
9. **Security first** - Non-root in production, secrets in env files, minimal images

Remember: **Two modes** - Development (fast iteration) and Production (validation/deployment). Both must work correctly.
