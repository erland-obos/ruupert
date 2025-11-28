# Docker Setup Guidelines

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should set
> up Docker environments when assisting with any project. They are not designed or supported for use with other AI agents
> or models.

## Overview

This guide defines how to containerize applications, supporting both **fast local development** with hot-reload and *
*production-ready deployments**. The setup uses Docker Compose for orchestration and can be adapted for both single
applications and monorepo structures.

## Philosophy

- **Two modes**: Development (hot-reload, volume mounts) and Production (optimized, immutable)
- **Dev/prod parity**: Same dependencies and services in both environments
- **Simple orchestration**: Docker Compose for all multi-container scenarios
- **Security by default**: Non-root users, minimal images, no secrets in code
- **Flexible architecture**: Supports both single-app and multi-app setups

## When to Use Docker

### Development Mode (Fast Iteration)

- **Daily development** with hot-reload and instant feedback
- Run entire stack locally: `docker compose up`
- Changes to source code instantly reflected in running containers
- Full dev/prod parity

### Production Validation (Before Release)

- **CI/CD pipelines** and automated testing
- **Pre-release validation** to test production builds locally
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
├── .env.docker                     # Shared config - gitignored
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
   cp .env.docker.example .env.docker
```

2. **Edit environment files** with your configuration

3. **Start development environment**:

```bash
   docker compose up
```

### Daily Development

```bash
# Start all services with hot-reload
docker compose up

# Rebuild after dependency changes
docker compose up --build

# View logs
docker compose logs -f <service-name>

# Stop everything
docker compose down
```

### Production Validation

```bash
# Test production build locally
docker compose -f docker-compose.prod.yml up --build
```

## Dockerfile Template

### Multi-Stage Dockerfile Pattern

```dockerfile
# =============================================================================
# Stage: base - Shared base configuration
# =============================================================================
FROM <base-image> AS base

# Install runtime dependencies
# Set working directory
WORKDIR /app

# =============================================================================
# Stage: dependencies - Install all dependencies
# =============================================================================
FROM base AS dependencies

# Copy dependency manifests
COPY <dependency-files> ./

# Install ALL dependencies (including dev dependencies for development)
RUN <install-command>

# =============================================================================
# Stage: development - Hot-reload development environment
# =============================================================================
FROM dependencies AS development

# Copy source code
COPY src ./src
COPY <config-files> ./

# Development uses watch mode
CMD ["<dev-command>"]

# =============================================================================
# Stage: build - Build production bundle
# =============================================================================
FROM dependencies AS build

# Copy source code and build configs
COPY src ./src
COPY <build-configs> ./

# Build the application
RUN <build-command>

# =============================================================================
# Stage: production - Minimal production runtime
# =============================================================================
FROM <minimal-base-image> AS production

# Create non-root user for security
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

# Copy dependency manifests
COPY --chown=appuser:appuser <dependency-files> ./

# Install production dependencies only
RUN <install-prod-command>

# Copy built application from build stage
COPY --chown=appuser:appuser --from=build /app/dist ./dist

# Switch to non-root user
USER appuser

# Set production environment
ENV NODE_ENV=production

# Health check (if applicable)
# HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
#   CMD <health-check-command>

# Start the application
CMD ["<start-command>"]
```

### Key Points

- **Four stages**: `base`, `dependencies`, `development`, `production`
- **Development target**: Includes dev dependencies, runs watch mode
- **Production target**: Minimal, optimized, non-root user
- **Selective copy**: Only necessary files copied to each stage

## .dockerignore Template

```
# Dependencies (installed in container)
node_modules
vendor
.venv
__pycache__

# Build output (generated in container)
dist
build
coverage
*.log

# Development files
.env*
!.env.docker.example
*.test.*
**/__tests__
**/__mocks__

# IDE and OS files
.idea
.vscode
.DS_Store
*.swp
Thumbs.db

# Git
.git
.gitignore

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

```yaml
version: '3.9'

services:
  app-dev:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    container_name: <project-name>-app-dev
    restart: unless-stopped
    volumes:
      # Mount source code for hot-reload
      - ./src:/app/src:cached
      # Mount config files (read-only)
      - ./<config-file>:/app/<config-file>:ro
    environment:
      - APP_ENV=development
    env_file:
      - .env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network
    # ports:
    #   - "3000:3000"

  db:
    image: <database-image>
    container_name: <project-name>-db
    restart: unless-stopped
    environment:
      # Database-specific environment variables
    volumes:
      - db-data:/var/lib/<db-data-path>
    ports:
      - "<host-port>:<container-port>"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "<health-check-command>"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

### Production: `docker-compose.prod.yml`

```yaml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    image: <project-name>-app:latest
    container_name: <project-name>-app
    restart: unless-stopped
    environment:
      - APP_ENV=production
    env_file:
      - .env.docker
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: <database-image>
    container_name: <project-name>-db
    restart: unless-stopped
    environment:
      # Use secrets from environment
    volumes:
      - db-data:/var/lib/<db-data-path>
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "<health-check-command>"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

## Environment Variables

### Template: `.env.docker.example`

```bash
# =============================================================================
# Docker Environment Variables
# Copy this file to .env.docker and customize
# =============================================================================

# Database
DB_PASSWORD=changeme_in_production
DB_USER=app
DB_NAME=appdb

# Application
LOG_LEVEL=info

# External Services (if any)
# EXTERNAL_API_KEY=your_api_key_here
```

## Common Commands

### Development Workflow

```bash
# Start all services
docker compose up

# Start in background
docker compose up -d

# Rebuild after dependency changes
docker compose up --build <service>

# View logs
docker compose logs -f <service>

# Execute commands in running container
docker compose exec <service> <command>

# Stop all services
docker compose down

# Stop and remove volumes (fresh start)
docker compose down -v
```

### Production Validation

```bash
# Build and run production images
docker compose -f docker-compose.prod.yml up --build

# Check logs
docker compose -f docker-compose.prod.yml logs -f <service>

# Stop
docker compose -f docker-compose.prod.yml down
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
│  ┌────────┐  ┌────────┐ │
│  │  app   │  │   db   │ │
│  └────────┘  └────────┘ │
│                         │
└─────────────────────────┘
```

### Monorepo: Isolated Networks with Shared Services

```
┌─────────────────────────────────────────────────┐
│              shared-services                    │
│                                                 │
│  ┌──────────────────────────────────────────┐  │
│  │         Database / Cache / etc.          │  │
│  └──────────────────────────────────────────┘  │
│                                                 │
└────────┬─────────────────────────────┬──────────┘
         │                             │
┌────────┴─────────┐         ┌────────┴─────────┐
│  app1-network    │         │  app2-network    │
│  ┌────────────┐  │         │  ┌────────────┐  │
│  │   app1     │  │         │  │   app2     │  │
│  └────────────┘  │         │  └────────────┘  │
└──────────────────┘         └──────────────────┘
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
     - ./config.yaml:/app/config.yaml:ro
```

3. **Dependencies (not mounted)**: Let container manage them

### Performance Optimization

On macOS and Windows, use `:cached` for better performance:

```yaml
volumes:
  - ./src:/app/src:cached
```

## Troubleshooting

### Build Fails

```bash
# Rebuild without cache
docker compose build --no-cache <service>

# Check logs
docker compose logs <service>
```

### Hot Reload Not Working

```bash
# Check volume mounts
docker compose exec <service> ls -la /app/src

# Restart the service
docker compose restart <service>
```

### Database Connection Refused

```bash
# Check database is healthy
docker compose ps db

# Check logs
docker compose logs db

# Test connection from app container
docker compose exec <service> <connection-test-command>
```

### Container Exits Immediately

```bash
# Check logs for errors
docker compose logs <service>

# Run with interactive shell to debug
docker compose run --rm <service> sh
```

### Port Already in Use

```bash
# Find process using the port
lsof -i :<port>

# Change port in docker-compose.yml
ports:
  - "<different-port>:<container-port>"
```

## Security Best Practices

### 1. Non-Root User (Production)

```dockerfile
USER appuser
```

### 2. Environment Variables

- Never commit secrets
- Use `.env.docker` (gitignored) for actual values
- Use `.env.docker.example` as template (committed)

### 3. Network Isolation

- Use separate networks per app
- Only expose necessary ports

### 4. Read-Only Mounts

```yaml
volumes:
  - ./config:/app/config:ro
```

### 5. Minimal Base Images

Use slim/alpine variants when possible.

### 6. Pin Versions

```dockerfile
FROM node:22.0.0-slim  # Not node:22-slim
```

## CI/CD Integration

### Example Workflow Steps

1. **Set up environment files** from secrets
2. **Build development images**
3. **Run tests in container**
4. **Build production image**
5. **Test production image**
6. **Push to registry** (on main branch)

## Performance Tips

### 1. Layer Caching

Order Dockerfile instructions from least to most frequently changed:

```dockerfile
# Good: stable layers first
COPY <lock-file> ./
RUN <install-command>
COPY src ./src
RUN <build-command>
```

### 2. Multi-Stage Builds

Always use multi-stage builds to minimize final image size.

### 3. Prune Regularly

```bash
docker system prune -a
```

## Checklist: Dockerizing an Application

- [ ] Create `Dockerfile` from template
- [ ] Create `.dockerignore`
- [ ] Replace all placeholders with actual names
- [ ] Create `.env.docker.example` with required variables
- [ ] Create `.env.docker` and add to `.gitignore`
- [ ] Create `docker-compose.yml` (development)
- [ ] Create `docker-compose.prod.yml` (production)
- [ ] Test development build: `docker compose up --build`
- [ ] Verify hot-reload works
- [ ] Test production build: `docker compose -f docker-compose.prod.yml up --build`
- [ ] Test database connectivity
- [ ] Verify non-root user in production
- [ ] Check final image size: `docker images`

## Notes for Claude Code

When dockerizing an application:

1. **Use the templates** - Consistency is critical
2. **Replace all placeholders** - project names, database names, etc.
3. **Test both modes** - Development (hot-reload) and Production (validation)
4. **Verify hot-reload** - Make a code change and check it reflects immediately
5. **Validate database access** - Ensure connectivity works
6. **Document deviations** - If template doesn't work, document why
7. **Keep it simple** - Don't add complexity unless required
8. **Security first** - Non-root in production, secrets in env files, minimal images

Remember: **Two modes** - Development (fast iteration) and Production (validation/deployment). Both must work correctly.
