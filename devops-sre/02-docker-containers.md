# 2. Docker & Containers

## 📚 Quick Summary

Docker revolutionized software deployment - learn the technology powering 90% of cloud applications!

**What You'll Learn:**
- **Container Basics**: What containers are and why they matter
- **Docker Architecture**: Images, containers, registries
- **Dockerfile Mastery**: Build optimized images
- **Docker Networking**: Container communication
- **Docker Volumes**: Data persistence
- **Multi-stage Builds**: Production-ready images
- **Docker Compose**: Multi-container applications
- **Security**: Container scanning and best practices

**Why This Matters:**
- Kubernetes runs containers (Docker knowledge essential)
- Every modern application uses containers
- Local dev = Production (consistency)
- Faster deployments (seconds vs minutes)
- Interview questions: 15% are Docker-based

**Interview Reality:**
"How would you containerize this application?" = Daily DevOps task!

---

## 📖 Simple Explanation

**What is a Container?**
Think of a container like a shipping container for software:
- **Consistency**: Works same everywhere (laptop, server, cloud)
- **Isolation**: Doesn't interfere with other apps
- **Portable**: Move easily between environments
- **Lightweight**: Shares host OS, not full VM

**Container vs Virtual Machine:**
```
Virtual Machine:
┌─────────────────────┐
│   Application       │
│   Libraries         │
│   Guest OS          │  ← Full OS (GBs)
│   Hypervisor        │
│   Host OS           │
│   Hardware          │
└─────────────────────┘
Start time: Minutes
Size: GBs

Container:
┌─────────────────────┐
│   Application       │
│   Libraries         │
│   Container Runtime │  ← Shared OS
│   Host OS           │
│   Hardware          │
└─────────────────────┘
Start time: Seconds
Size: MBs
```

**Real-World Analogy:**
```
Before Docker:
Developer: "It works on my machine!"
Ops: "Well, it doesn't work in production!"

With Docker:
Developer: "Here's the container, it works!"
Ops: "Great, runs perfectly in production!"
```

---

## Table of Contents
- [Docker Fundamentals](#docker-fundamentals)
- [Docker Architecture](#docker-architecture)
- [Working with Images](#working-with-images)
- [Working with Containers](#working-with-containers)
- [Dockerfile Best Practices](#dockerfile-best-practices)
- [Docker Networking](#docker-networking)
- [Docker Volumes](#docker-volumes)
- [Docker Compose](#docker-compose)
- [Container Security](#container-security)
- [Production Best Practices](#production-best-practices)

---

## Docker Fundamentals

### 📖 Simple Explanation

**Core Concepts:**

**1. Image = Blueprint**
- Recipe for creating a container
- Read-only template
- Like a class in programming

**2. Container = Running Instance**
- Created from an image
- Running process
- Like an object in programming

**3. Registry = App Store**
- Stores Docker images
- Docker Hub (public)
- Private registries (AWS ECR, GCR, Harbor)

**Relationship:**
```
Image (stored) → Run → Container (active)
      ↓
   Push/Pull
      ↓
   Registry (Docker Hub)
```

---

### Installing Docker

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER
newgrp docker  # or logout/login

# Verify installation
docker --version
# Docker version 24.0.7

docker run hello-world
# Should download and run a test container
```

---

## Docker Architecture

### 📖 Simple Explanation

**Docker Architecture:**
```
┌──────────────────────────────────────┐
│         Docker Client (CLI)          │  ← You type commands here
│         docker run, docker build     │
└─────────────────┬────────────────────┘
                  │ Docker API
┌─────────────────▼────────────────────┐
│         Docker Daemon (Engine)       │  ← Does the work
│  - Manages containers                │
│  - Manages images                    │
│  - Manages networks                  │
│  - Manages volumes                   │
└──────────────────────────────────────┘
```

**Components:**
- **Docker Client**: CLI you interact with
- **Docker Daemon**: Background service doing work
- **Docker Registry**: Stores images (Docker Hub)
- **Images**: Read-only templates
- **Containers**: Running instances
- **Networks**: Container communication
- **Volumes**: Data persistence

---

## Working with Images

### 📖 Simple Explanation

**Images are layered:**
```
┌─────────────────────┐
│  Your Application   │  ← Layer 3 (your code)
├─────────────────────┤
│  Python 3.11        │  ← Layer 2 (runtime)
├─────────────────────┤
│  Ubuntu 22.04       │  ← Layer 1 (base OS)
└─────────────────────┘

Each layer is cached!
Change layer 3 → Only rebuild layer 3
```

---

### Image Commands

```bash
# Search for images
docker search nginx
docker search python

# Pull images from Docker Hub
docker pull nginx
docker pull nginx:1.25                # specific version
docker pull python:3.11-slim          # slim variant

# List local images
docker images
docker images -a                      # all images (including intermediate)

# Inspect image
docker image inspect nginx
docker image history nginx            # see layers

# Tag images
docker tag nginx:latest mynginx:v1.0

# Remove images
docker rmi nginx                      # remove by name
docker rmi abc123def                  # remove by ID
docker image prune                    # remove unused images
docker image prune -a                 # remove all unused

# Save/Load images (for air-gapped environments)
docker save nginx > nginx.tar
docker load < nginx.tar
```

**Real Example:**
```bash
# Pull Python image
$ docker pull python:3.11-slim
3.11-slim: Pulling from library/python
01b5b2efb836: Pull complete
a3246e691a30: Pull complete
42541d7eb6db: Pull complete
aae5f5b74f19: Pull complete
b4c35e3f8d54: Pull complete
Digest: sha256:abc123...
Status: Downloaded newer image for python:3.11-slim

# Verify
$ docker images
REPOSITORY   TAG          IMAGE ID       CREATED        SIZE
python       3.11-slim    1a2b3c4d5e6f   2 weeks ago    125MB
```

---

## Working with Containers

### 📖 Simple Explanation

**Container Lifecycle:**
```
Created → Running → Paused → Stopped → Removed
   ↑         ↓         ↓         ↓         ↓
 docker   docker    docker   docker   docker
 create    start     pause     stop      rm
```

---

### Container Commands

```bash
# Run a container
docker run nginx                      # runs and attaches to terminal
docker run -d nginx                   # detached (background)
docker run -d --name web nginx        # with custom name
docker run -d -p 8080:80 nginx        # port mapping (host:container)
docker run -d -p 8080:80 -v /data:/data nginx  # with volume

# Common options
docker run -d \
  --name myapp \                      # custom name
  -p 3000:3000 \                      # port mapping
  -e NODE_ENV=production \            # environment variable
  -v $(pwd)/data:/app/data \          # volume mount
  --restart unless-stopped \          # restart policy
  myapp:latest                        # image name

# List containers
docker ps                             # running containers
docker ps -a                          # all containers (including stopped)
docker ps -q                          # only IDs

# Container logs
docker logs myapp                     # view logs
docker logs -f myapp                  # follow logs (real-time)
docker logs --tail 100 myapp          # last 100 lines
docker logs --since 10m myapp         # logs from last 10 minutes

# Execute commands in container
docker exec myapp ls /app             # run command
docker exec -it myapp bash            # interactive shell
docker exec -it myapp sh              # if bash not available

# Start/Stop containers
docker start myapp                    # start stopped container
docker stop myapp                     # graceful stop (10s timeout)
docker kill myapp                     # force stop
docker restart myapp                  # restart container

# Pause/Unpause
docker pause myapp                    # pause processes
docker unpause myapp                  # resume

# Remove containers
docker rm myapp                       # remove stopped container
docker rm -f myapp                    # force remove (even if running)
docker container prune                # remove all stopped containers

# Inspect container
docker inspect myapp                  # detailed info (JSON)
docker top myapp                      # running processes
docker stats myapp                    # resource usage (CPU, memory)
docker stats                          # all containers

# Copy files
docker cp myapp:/app/log.txt ./       # from container to host
docker cp ./file.txt myapp:/app/      # from host to container
```

**Real Example: Run Nginx Web Server**
```bash
# Run Nginx on port 8080
$ docker run -d --name webserver -p 8080:80 nginx
a1b2c3d4e5f6...

# Verify it's running
$ docker ps
CONTAINER ID   IMAGE   COMMAND                  STATUS         PORTS
a1b2c3d4e5f6   nginx   "/docker-entrypoint.…"   Up 2 seconds   0.0.0.0:8080->80/tcp

# Test it
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title></head>
...

# View logs
$ docker logs webserver
...
172.17.0.1 - - [22/Oct/2024:10:30:00 +0000] "GET / HTTP/1.1" 200

# Access container shell
$ docker exec -it webserver bash
root@a1b2c3d4e5f6:/# ls /usr/share/nginx/html/
index.html
root@a1b2c3d4e5f6:/# exit

# Stop and remove
$ docker stop webserver
$ docker rm webserver
```

---

## Dockerfile Best Practices

### 📖 Simple Explanation

**Dockerfile = Recipe for building an image**

**Structure:**
```dockerfile
FROM base-image           # Start with this
WORKDIR /app             # Set working directory
COPY . .                 # Copy files
RUN commands             # Execute commands
EXPOSE 8080              # Document port
CMD ["start", "app"]     # Default command
```

---

### Basic Dockerfile

```dockerfile
# Simple Python application
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Copy requirements first (better caching)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 8000

# Run application
CMD ["python", "app.py"]
```

**Build and run:**
```bash
# Build image
docker build -t myapp:v1.0 .

# Run container
docker run -d -p 8000:8000 myapp:v1.0
```

---

### Multi-Stage Build (Production)

**Problem:** Development images are huge (includes build tools, test dependencies)

**Solution:** Multi-stage builds

```dockerfile
# Stage 1: Build
FROM node:18 AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Build application
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine AS production

WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

**Result:**
- Builder stage: 1.2GB (includes dev tools)
- Production stage: 180MB (only runtime + app)
- **85% size reduction!**

---

### Dockerfile Best Practices

```dockerfile
# ✅ GOOD Dockerfile

# 1. Use specific version tags (not latest)
FROM node:18.17-alpine

# 2. Use alpine for smaller size
# alpine = minimal Linux distro

# 3. Set working directory
WORKDIR /app

# 4. Copy package files first (better caching)
COPY package*.json ./

# 5. Install dependencies separately
RUN npm ci --only=production

# 6. Copy source code last (changes frequently)
COPY . .

# 7. Run as non-root user (security)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

USER nodejs

# 8. Expose port (documentation)
EXPOSE 3000

# 9. Health check (optional but good)
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node healthcheck.js || exit 1

# 10. Use ENTRYPOINT + CMD for flexibility
ENTRYPOINT ["node"]
CMD ["dist/main.js"]
```

**Common Dockerfile Instructions:**
```dockerfile
FROM image:tag          # Base image
WORKDIR /path           # Set working directory
COPY src dest           # Copy files from host
ADD url dest            # Copy + extract archives
RUN command             # Execute command during build
ENV KEY=value           # Set environment variable
EXPOSE port             # Document exposed port
USER username           # Set user for subsequent commands
VOLUME /path            # Create mount point
ENTRYPOINT ["cmd"]      # Container executable
CMD ["arg1", "arg2"]    # Default arguments
LABEL key=value         # Add metadata
ARG name=default        # Build-time variable
HEALTHCHECK CMD         # Health check command
```

---

### .dockerignore

```
# .dockerignore (like .gitignore for Docker)

# Version control
.git
.gitignore

# Dependencies
node_modules
vendor

# Build artifacts
dist
build
*.o
*.so

# Logs
*.log
logs/

# IDE
.vscode
.idea
*.swp

# Test files
test/
*.test.js
coverage/

# Documentation
*.md
docs/

# CI/CD
.github
.gitlab-ci.yml

# Environment files
.env
.env.local
```

---

## Docker Networking

### 📖 Simple Explanation

**Docker networks allow containers to communicate:**

**Network Types:**
1. **Bridge** (default): Containers on same host
2. **Host**: Container uses host's network directly
3. **None**: No networking
4. **Custom**: User-defined networks (recommended)

```
┌─────────────────────────────────────────┐
│            Host Machine                 │
│                                         │
│  ┌──────────────────────────────┐      │
│  │     Bridge Network (default) │      │
│  │                              │      │
│  │   ┌─────────┐   ┌─────────┐ │      │
│  │   │Container│   │Container│ │      │
│  │   │   Web   │<->│   DB    │ │      │
│  │   └─────────┘   └─────────┘ │      │
│  └──────────────────────────────┘      │
│            ↕                            │
│        Host Network                     │
└─────────────────────────────────────────┘
```

---

### Network Commands

```bash
# List networks
docker network ls

# Create custom network
docker network create mynetwork
docker network create --driver bridge mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork container1

# Disconnect from network
docker network disconnect mynetwork container1

# Remove network
docker network rm mynetwork

# Prune unused networks
docker network prune
```

**Real Example: Web + Database**
```bash
# Create custom network
$ docker network create app-network

# Run PostgreSQL database
$ docker run -d \
  --name postgres \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Run web application
$ docker run -d \
  --name webapp \
  --network app-network \
  -p 8080:8080 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/db \
  mywebapp:latest

# Containers can communicate by name!
# webapp connects to "postgres" (DNS resolution)
```

**Port Mapping:**
```bash
# Format: -p HOST_PORT:CONTAINER_PORT

# Single port
docker run -d -p 8080:80 nginx

# Multiple ports
docker run -d -p 80:80 -p 443:443 nginx

# Random host port
docker run -d -p 80 nginx  # Docker assigns random host port

# Specific IP
docker run -d -p 127.0.0.1:8080:80 nginx  # only localhost

# All ports
docker run -d -P nginx  # Map all EXPOSE ports to random host ports
```

---

## Docker Volumes

### 📖 Simple Explanation

**Problem:** Container deleted = Data lost!  
**Solution:** Volumes = Persistent storage

**Types:**
1. **Volume** (managed by Docker) - Recommended
2. **Bind Mount** (map host directory)
3. **tmpfs** (memory only, temporary)

```
Container is deleted → Data in volume remains!

┌─────────────────────┐
│   Container         │
│   /app/data    ─────┼──→ Volume (persists)
└─────────────────────┘     /var/lib/docker/volumes/
```

---

### Volume Commands

```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove unused volumes
docker volume prune
```

**Using Volumes:**
```bash
# Named volume
docker run -d \
  --name db \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# Bind mount (development)
docker run -d \
  --name webapp \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  node:18

# Read-only mount
docker run -d \
  -v myconfig:/config:ro \
  myapp

# tmpfs (sensitive data, in-memory)
docker run -d \
  --tmpfs /tmp \
  myapp
```

**Real Example: Database with Persistence**
```bash
# Create volume
$ docker volume create pgdata

# Run PostgreSQL with volume
$ docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Add data
$ docker exec -it postgres psql -U postgres -c \
  "CREATE TABLE users (id INT, name TEXT);"

# Stop and remove container
$ docker stop postgres
$ docker rm postgres

# Run new container with same volume
$ docker run -d \
  --name postgres-new \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Data is still there!
$ docker exec -it postgres-new psql -U postgres -c \
  "SELECT * FROM users;"
# Data persists!
```

---

## Docker Compose

### 📖 Simple Explanation

**Docker Compose = Multi-container applications as code**

**Instead of:**
```bash
docker network create mynetwork
docker run -d --name db --network mynetwork ...
docker run -d --name redis --network mynetwork ...
docker run -d --name web --network mynetwork ...
# 10+ commands!
```

**Use docker-compose.yml:**
```bash
docker compose up -d
# One command to start everything!
```

---

### Docker Compose File

```yaml
# docker-compose.yml

version: '3.8'

services:
  # Web application
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped
    volumes:
      - ./src:/app/src  # for development
    networks:
      - app-network

  # PostgreSQL database
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    networks:
      - app-network

# Named volumes
volumes:
  postgres-data:

# Custom network
networks:
  app-network:
    driver: bridge
```

---

### Docker Compose Commands

```bash
# Start services
docker compose up                     # foreground
docker compose up -d                  # detached (background)
docker compose up --build             # rebuild images first

# Stop services
docker compose stop                   # stop (containers remain)
docker compose down                   # stop and remove containers
docker compose down -v                # also remove volumes

# View status
docker compose ps                     # running services
docker compose ps -a                  # all services

# View logs
docker compose logs                   # all services
docker compose logs web               # specific service
docker compose logs -f                # follow
docker compose logs -f --tail=100 web # last 100 lines

# Execute commands
docker compose exec web bash          # interactive shell
docker compose exec db psql -U postgres  # run command

# Scale services
docker compose up -d --scale web=3    # run 3 web containers

# Restart services
docker compose restart                # all services
docker compose restart web            # specific service

# Build images
docker compose build                  # build all
docker compose build web              # build specific service

# Pull images
docker compose pull                   # pull all images

# Validate compose file
docker compose config                 # check syntax
docker compose config --quiet         # just validate
```

**Real Example: Full Stack Application**
```yaml
# docker-compose.yml - Complete web app

version: '3.8'

services:
  # Frontend (React)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8080
    volumes:
      - ./frontend/src:/app/src
    depends_on:
      - backend

  # Backend (Node.js API)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://postgres:secret@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=my-secret-key
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  # Database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  # Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  # Nginx (reverse proxy)
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend

volumes:
  postgres-data:
  redis-data:
```

**Usage:**
```bash
# Start everything
$ docker compose up -d
[+] Running 5/5
 ✔ Container app-postgres-1  Started
 ✔ Container app-redis-1     Started
 ✔ Container app-backend-1   Started
 ✔ Container app-frontend-1  Started
 ✔ Container app-nginx-1     Started

# Check status
$ docker compose ps
NAME                IMAGE              STATUS
app-postgres-1      postgres:15-alpine Up 10 seconds
app-redis-1         redis:7-alpine     Up 10 seconds
app-backend-1       app-backend        Up 5 seconds
app-frontend-1      app-frontend       Up 5 seconds
app-nginx-1         nginx:alpine       Up 2 seconds

# View logs
$ docker compose logs -f backend

# Access database
$ docker compose exec postgres psql -U postgres -d myapp

# Stop everything
$ docker compose down
```

---

## Container Security

### 📖 Simple Explanation

**Security = Defense in layers**

**Key Principles:**
1. **Minimal base images** (alpine, distroless)
2. **Non-root user** (don't run as root)
3. **Scan for vulnerabilities** (Trivy, Snyk)
4. **Least privilege** (only needed permissions)
5. **Secrets management** (not in images!)

---

### Security Best Practices

```dockerfile
# ✅ SECURE Dockerfile

# 1. Use minimal base image
FROM node:18-alpine

# 2. Update packages (security patches)
RUN apk update && apk upgrade

# 3. Install only what's needed
RUN apk add --no-cache dumb-init

WORKDIR /app

# 4. Copy files with specific permissions
COPY --chown=node:node package*.json ./

# 5. Don't run as root!
USER node

RUN npm ci --only=production

COPY --chown=node:node . .

# 6. Use dumb-init (proper signal handling)
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "server.js"]
```

**Security Scanning:**
```bash
# Scan image for vulnerabilities (Trivy)
$ docker scan myimage:latest
$ trivy image myimage:latest

# Example output:
myimage:latest (alpine 3.18.0)
================================
Total: 15 (UNKNOWN: 0, LOW: 5, MEDIUM: 8, HIGH: 2, CRITICAL: 0)

HIGH: CVE-2023-1234
Package: openssl
Installed Version: 3.0.7-r0
Fixed Version: 3.0.8-r0
```

**Docker Bench Security:**
```bash
# Run Docker security audit
$ docker run --rm --net host --pid host --userns host --cap-add audit_control \
    -v /var/lib:/var/lib \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /etc:/etc:ro \
    docker/docker-bench-security
```

---

### Security Checklist

```bash
# ✅ 1. Use specific image versions
FROM node:18.17-alpine  # ✅ Specific
FROM node:18            # ⚠️ Less specific
FROM node:latest        # ❌ Unpredictable

# ✅ 2. Scan images
trivy image myapp:latest
docker scan myapp:latest

# ✅ 3. Run as non-root
USER node  # or create custom user

# ✅ 4. Use read-only filesystem where possible
docker run --read-only myapp

# ✅ 5. Drop unnecessary capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# ✅ 6. Set resource limits
docker run -m 512m --cpus=1 myapp

# ✅ 7. Use secrets management
docker secret create db_password db_password.txt
docker service create --secret db_password myapp

# ✅ 8. Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1
docker pull myapp  # Only pulls signed images

# ✅ 9. Use .dockerignore
# Prevent secrets from being added to image

# ✅ 10. Regular updates
docker pull nginx:latest
docker build --no-cache -t myapp:latest .
```

---

## Production Best Practices

### Image Optimization

```dockerfile
# ❌ BAD: Large image (1.2GB)
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    python3 python3-pip curl wget vim
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY . .
CMD ["python3", "app.py"]

# ✅ GOOD: Optimized image (180MB)
FROM python:3.11-alpine

# Install system dependencies in one layer
RUN apk add --no-cache \
    libpq \
    && rm -rf /var/cache/apk/*

WORKDIR /app

# Copy and install requirements separately (caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Non-root user
RUN adduser -D appuser && chown -R appuser /app
USER appuser

CMD ["python", "app.py"]
```

**Size Comparison:**
- Ubuntu base: 77MB
- Debian slim: 27MB
- Alpine: 7MB
- Distroless: 2MB

---

### Production Dockerfile Template

```dockerfile
# Multi-stage build for production

# ============================================
# Stage 1: Builder
# ============================================
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Build application
COPY . .
RUN npm run build && \
    npm run test

# ============================================
# Stage 2: Dependencies
# ============================================
FROM node:18-alpine AS dependencies

WORKDIR /app

# Install only production dependencies
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# ============================================
# Stage 3: Production
# ============================================
FROM node:18-alpine AS production

# Install dumb-init
RUN apk add --no-cache dumb-init

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy production dependencies
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js || exit 1

# Use dumb-init for proper signal handling
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "dist/main.js"]
```

---

## Common Pitfalls

### ❌ Mistake 1: Running as Root
```dockerfile
# Bad
FROM node:18
WORKDIR /app
COPY . .
CMD ["node", "app.js"]  # Runs as root!

# Good
FROM node:18
WORKDIR /app
COPY --chown=node:node . .
USER node  # ✅ Non-root
CMD ["node", "app.js"]
```

### ❌ Mistake 2: Secrets in Images
```dockerfile
# Bad - Secret in image!
ENV DATABASE_PASSWORD=supersecret

# Good - Pass at runtime
docker run -e DATABASE_PASSWORD=supersecret myapp
# Or use Docker secrets
docker secret create db_pass password.txt
```

### ❌ Mistake 3: Not Using .dockerignore
```
# Without .dockerignore:
# node_modules, .git, logs all copied → huge image

# With .dockerignore:
node_modules
.git
*.log
.env
```

### ❌ Mistake 4: Unnecessary Packages
```dockerfile
# Bad - 500MB of unnecessary tools
RUN apt-get install -y curl wget vim nano htop

# Good - Only what's needed
RUN apk add --no-cache libpq
```

---

## Interview Tips

### 🎯 Most Asked Questions

**Q1: Docker vs VM - what's the difference?**
```
VM:
- Full OS per instance
- Hypervisor overhead
- GBs of space
- Minutes to start
- Hardware-level isolation

Container:
- Shared OS kernel
- No hypervisor
- MBs of space
- Seconds to start
- Process-level isolation

Use case:
- VM: Need different OS, strong isolation
- Container: Microservices, rapid scaling
```

**Q2: Explain Docker architecture**
```
Client → API → Daemon

Client: docker CLI
Daemon: dockerd (engine)
Registry: Docker Hub (images)

docker run:
1. Client sends command to daemon
2. Daemon checks local images
3. If not found, pulls from registry
4. Creates container from image
5. Starts container
```

**Q3: How do containers communicate?**
```
1. Bridge network (default)
   - Containers on same host
   - Access by container name

2. Host network
   - Share host's network
   - No isolation

3. Custom network
   - User-defined
   - Better DNS resolution

Example:
docker network create mynet
docker run --network mynet --name web nginx
docker run --network mynet --name db postgres
# web can access db by name: postgres:5432
```

**Q4: What's a multi-stage build?**
```
Multiple FROM statements in Dockerfile.

Benefits:
- Smaller final image
- Separate build and runtime dependencies
- More secure (no build tools in production)

Example:
FROM node:18 AS builder  # Build stage
RUN npm run build

FROM node:18-alpine      # Production stage
COPY --from=builder /app/dist ./dist
```

**Q5: How do you optimize Docker images?**
```
1. Use alpine/distroless base images
2. Multi-stage builds
3. Layer caching (.dockerignore, order matters)
4. Combine RUN commands
5. Remove package manager cache
6. Use specific versions (not latest)
7. Scan for vulnerabilities

Example:
FROM python:3.11-alpine  # ✅ Alpine (7MB vs 77MB)
RUN apk add --no-cache libpq && \  # ✅ Combined
    rm -rf /var/cache/apk/*         # ✅ Clean cache
```

---

## Real-World Scenario

### Dockerizing a Node.js Application

**Scenario:** Production-ready containerization

**1. Application Structure:**
```
myapp/
├── src/
│   └── server.js
├── package.json
├── package-lock.json
├── Dockerfile
├── .dockerignore
└── docker-compose.yml
```

**2. Dockerfile (Production-Ready):**
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine

RUN apk add --no-cache dumb-init && \
    addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

COPY --from=builder --chown=nodejs:nodejs /app/package*.json ./
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s CMD node -e 'require("http").get("http://localhost:3000/health", (r) => process.exit(r.statusCode === 200 ? 0 : 1))'

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server.js"]
```

**3. .dockerignore:**
```
node_modules
npm-debug.log
.git
.env
dist
.vscode
.idea
*.md
test/
coverage/
```

**4. docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:secret@db:5432/myapp
    depends_on:
      - db
      - redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

**5. Deployment:**
```bash
# Build
docker compose build

# Start
docker compose up -d

# Check health
docker compose ps
docker compose logs app

# Scale if needed
docker compose up -d --scale app=3

# Update
docker compose build
docker compose up -d --no-deps app  # Only update app

# Cleanup
docker compose down -v
```

---

## Chapter Summary

### What You Learned

**1. Docker Fundamentals**
- ✅ Containers vs VMs
- ✅ Images, containers, registries
- ✅ Docker architecture (client, daemon, registry)

**2. Working with Docker**
- ✅ Pull, run, stop containers
- ✅ docker run with options
- ✅ Logs, exec, inspect commands

**3. Dockerfile Mastery**
- ✅ Write efficient Dockerfiles
- ✅ Multi-stage builds
- ✅ Layer optimization
- ✅ .dockerignore usage

**4. Networking & Volumes**
- ✅ Bridge, host, custom networks
- ✅ Container communication
- ✅ Persistent storage with volumes

**5. Docker Compose**
- ✅ Multi-container applications
- ✅ docker-compose.yml syntax
- ✅ Service orchestration

**6. Security**
- ✅ Non-root users
- ✅ Image scanning
- ✅ Minimal base images
- ✅ Secrets management

---

### 🎯 Interview Quick Tips

**Must-Know Commands:**
```bash
docker build -t app .
docker run -d -p 8080:80 nginx
docker ps
docker logs -f container
docker exec -it container bash
docker compose up -d
```

**Key Concepts:**
- Image = Template, Container = Running instance
- Layers are cached for efficiency
- Multi-stage builds reduce image size
- Volumes persist data
- Networks enable communication
- Compose manages multi-container apps

---

### 💡 Real-World Tips

**Daily Workflow:**
```bash
# Development
docker compose up        # Start all services
docker compose logs -f   # Watch logs
docker compose exec app bash  # Debug

# Cleanup
docker system prune -a   # Clean everything (careful!)
docker volume prune      # Clean volumes
```

**Production Checklist:**
- [ ] Use specific image versions (not latest)
- [ ] Multi-stage builds for optimization
- [ ] Run as non-root user
- [ ] Add health checks
- [ ] Scan for vulnerabilities
- [ ] Use .dockerignore
- [ ] Set resource limits
- [ ] Configure restart policies

---

### 📚 What's Next?

Now that you know Docker, you're ready for:
- **Kubernetes** (orchestrates containers at scale)
- **CI/CD** (build and deploy Docker images automatically)
- **Cloud platforms** (ECS, EKS, GKE - all use containers)

**Practice Suggestions:**
1. Containerize your own application
2. Write a multi-stage Dockerfile
3. Create docker-compose for full-stack app
4. Scan images for vulnerabilities
5. Deploy to cloud (AWS ECS, GCP Cloud Run)

---

**Next Chapter:** [Kubernetes Fundamentals](03-kubernetes-fundamentals.md)

**Prerequisites Covered:** ✅ Ready for Kubernetes!

