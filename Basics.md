# Docker Basics

## 📌 Overview

Docker is a containerization platform that packages applications and dependencies into isolated containers for consistent deployment across environments. It enables lightweight, portable, and scalable application delivery.

### **Why Docker?**

- **Consistency** - Same environment across dev, test, and production
- **Isolation** - Applications run in isolated containers
- **Efficiency** - Lightweight compared to virtual machines
- **Scalability** - Easy to scale horizontally
- **Portability** - Run anywhere Docker is installed
- **Rapid Deployment** - Quick startup times
- **Microservices** - Perfect for containerized architectures

------------------------------------------------------------------------

## Core Concepts

### **What is an Image?**

A Docker image is a lightweight, standalone, executable package that contains everything needed to run an application:
- Base operating system (lightweight, like Alpine Linux)
- Runtime environment (Node.js, Python, Java, etc.)
- Application code
- Dependencies and libraries
- Environment variables and configuration

**Key characteristics:**
- **Immutable** - Once built, it doesn't change
- **Layered** - Built in layers, each layer is a file system change
- **Portable** - Same image runs the same everywhere
- **Reusable** - Can be used to create multiple containers
- **Versioned** - Tagged with version numbers (e.g., myapp:1.0)

**Example:** Think of an image like a blueprint or template for a house - it contains all the specifications but isn't a physical house yet.

```
Image Components:
┌─────────────────────────────┐
│     Docker Image            │
├─────────────────────────────┤
│  Application Code           │
│  Dependencies/Libraries     │
│  Runtime (Node/Python/etc)  │
│  Base OS (Alpine/Ubuntu)    │
└─────────────────────────────┘
```

### **What is a Container?**

A Docker container is a running instance of an image - an actual isolated process/application running on your system.

**Key characteristics:**
- **Isolated** - Containers don't interfere with each other
- **Lightweight** - Shares OS kernel, no need for full OS per container
- **Ephemeral** - Can be created/destroyed quickly
- **Running instance** - An image becomes a container when executed
- **Stateless** - Typically designed to be disposable

**Example:** If an image is like a blueprint, a container is like an actual built house where people live.

```
Same Image → Multiple Containers

Image: nginx:latest
  ↓
┌──────────────┬──────────────┬──────────────┐
│ Container 1  │ Container 2  │ Container 3  │
│ (Port 8080)  │ (Port 8081)  │ (Port 8082)  │
└──────────────┴──────────────┴──────────────┘

All 3 containers run the same image but are separate instances
```

### **Images vs Containers - Quick Comparison**

| Aspect | Image | Container |
|--------|-------|-----------|
| **State** | Immutable (doesn't change) | Mutable (can change) |
| **Created from** | Dockerfile | Image |
| **Status** | Blueprint/Template | Running instance |
| **Persistence** | Persistent (stays until deleted) | Temporary (can be deleted) |
| **Storage** | Stored on disk | Runs in memory (uses image layers) |
| **Size** | Smaller (compressed) | Can grow (writes create layers) |
| **Quantity** | One image | Multiple containers from same image |

```
Dockerfile → Build → Image → Run → Container
  (recipe)   (cook)  (meal)  (eat) (consumption)
```

### **Images**

Blueprint for creating containers - immutable and layered:

```
Image Layers:
├── Base Layer (OS)
├── Dependencies Layer
├── Application Layer
└── Configuration Layer
```

### **Containers**

Running instances of images - lightweight and isolated:

```
Container = Image + Runtime Environment
```

### **Registries**

Repositories storing images:

- **Docker Hub** - Public registry
- **Amazon ECR** - AWS registry
- **Azure Container Registry** - Azure registry
- **Private registries** - Self-hosted

### **Dockerfile**

Blueprint for building images with instructions:

```dockerfile
# Base image
FROM ubuntu:22.04

# Metadata
LABEL maintainer="user@example.com"

# Install dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    pip

# Set working directory
WORKDIR /app

# Copy files
COPY requirements.txt .

# Install Python dependencies
RUN pip install -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 5000

# Environment variables
ENV FLASK_APP=app.py

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import requests; requests.get('http://localhost:5000')"

# Entrypoint
ENTRYPOINT ["python3"]

# Default command
CMD ["app.py"]
```

------------------------------------------------------------------------

## Dockerfile Instructions

| Instruction | Purpose |
|-------------|---------|
| FROM | Base image |
| RUN | Execute commands |
| COPY | Copy files from host |
| ADD | Copy files (with URL support) |
| WORKDIR | Set working directory |
| ENV | Set environment variables |
| EXPOSE | Document exposed ports |
| VOLUME | Define mount points |
| USER | Set user for commands |
| CMD | Default container command |
| ENTRYPOINT | Configure container as executable |
| HEALTHCHECK | Define health check |
| ARG | Build-time variables |

------------------------------------------------------------------------

## Building Images

### **Build from Dockerfile**

```bash
# Basic build
docker build -t myimage:1.0 .

# Build with build args
docker build -t myimage:1.0 \
  --build-arg NODE_ENV=production \
  --build-arg API_KEY=secret \
  .

# Build from URL
docker build -t myimage:1.0 https://github.com/user/repo.git

# Build specific Dockerfile
docker build -t myimage:1.0 -f Dockerfile.prod .

# Build with custom tags
docker build -t myimage:1.0 -t myimage:latest .
```

### **Image Optimization**

```dockerfile
# Multi-stage build to reduce size
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

------------------------------------------------------------------------

## Running Containers

### **Basic Container Operations**

```bash
# Run container
docker run -d --name mycontainer myimage:1.0

# Run with port mapping
docker run -d -p 8080:80 --name mycontainer myimage:1.0

# Run with volume mount
docker run -d -v /host/path:/container/path --name mycontainer myimage:1.0

# Run with environment variables
docker run -d -e DB_HOST=localhost -e DB_PORT=5432 --name mycontainer myimage:1.0

# Run interactive
docker run -it myimage:1.0 bash

# Run with resource limits
docker run -d --memory=512m --cpus=0.5 --name mycontainer myimage:1.0

# Run with network
docker run -d --network my-network --name mycontainer myimage:1.0

# Run with restart policy
docker run -d --restart=always --name mycontainer myimage:1.0

# List containers
docker ps                    # Running containers
docker ps -a                 # All containers

# View container logs
docker logs mycontainer      # View logs
docker logs -f mycontainer   # Follow logs
docker logs --tail 100 mycontainer  # Last 100 lines

# Stop container
docker stop mycontainer

# Start container
docker start mycontainer

# Restart container
docker restart mycontainer

# Remove container
docker rm mycontainer        # Remove stopped container
docker rm -f mycontainer     # Force remove running container

# Execute command in container
docker exec -it mycontainer bash
docker exec mycontainer python script.py

# View container info
docker inspect mycontainer

# View resource usage
docker stats mycontainer
docker stats               # All containers
```

------------------------------------------------------------------------

## Working with Images

```bash
# List images
docker images
docker images -a          # Including intermediate images
docker images --filter "dangling=true"  # Unused images

# Pull image from registry
docker pull ubuntu:22.04
docker pull myregistry.azurecr.io/myimage:latest

# Push image to registry
docker push myregistry.azurecr.io/myimage:latest

# Tag image
docker tag myimage:1.0 myimage:latest
docker tag myimage:1.0 myregistry.azurecr.io/myimage:1.0

# Remove image
docker rmi myimage:1.0
docker rmi -f myimage:1.0   # Force remove

# Search images
docker search python

# View image history
docker history myimage:1.0

# View image details
docker inspect myimage:1.0

# Save image to file
docker save -o myimage.tar myimage:1.0

# Load image from file
docker load -i myimage.tar
```

------------------------------------------------------------------------

## Networks

### **Network Types**

- **Bridge** - Default, isolated from host network
- **Host** - Share host network
- **None** - No network access
- **Overlay** - For Docker Swarm/Kubernetes

### **Network Commands**

```bash
# Create network
docker network create my-network

# Run container on network
docker run -d --network my-network --name app myimage:1.0

# Connect container to network
docker network connect my-network mycontainer

# Disconnect container from network
docker network disconnect my-network mycontainer

# List networks
docker network ls

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network
```

------------------------------------------------------------------------

## Volumes & Storage

### **Volume Types**

```bash
# Named volumes
docker volume create my-volume
docker run -d -v my-volume:/app/data myimage:1.0
docker volume ls
docker volume inspect my-volume
docker volume rm my-volume

# Bind mounts
docker run -d -v /host/path:/container/path myimage:1.0

# Tmpfs mounts (temporary, in memory)
docker run -d --tmpfs /app/temp myimage:1.0
```

### **Backup & Restore**

```bash
# Backup volume
docker run --rm -v my-volume:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/backup.tar.gz /data

# Restore volume
docker run --rm -v my-volume:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/backup.tar.gz -C /
```

------------------------------------------------------------------------

## Container Linking & Communication

### **Link Containers**

```bash
# Old method (deprecated)
docker run -d --name db postgres:latest
docker run -d --link db:database myapp:1.0

# New method (user-defined networks)
docker network create my-network
docker run -d --network my-network --name db postgres:latest
docker run -d --network my-network --name app myapp:1.0
# Inside app, connect to db using hostname: db:5432
```

------------------------------------------------------------------------

## Best Practices

### **Dockerfile Best Practices**

- **Use specific base image tags** - Avoid `latest`
- **Minimize layers** - Combine RUN commands
- **Use .dockerignore** - Exclude unnecessary files
- **Run as non-root user** - Improve security
- **Keep images small** - Use Alpine base images
- **Use multi-stage builds** - Reduce final image size
- **Leverage layer caching** - Order commands efficiently
- **Health checks** - Define HEALTHCHECK

### **Container Best Practices**

- **One process per container** - Follow single responsibility
- **Use environment variables** - For configuration
- **Log to stdout/stderr** - For easy access
- **Don't run as root** - Security best practice
- **Use resource limits** - Prevent resource exhaustion
- **Use named volumes** - For persistent data
- **Implement graceful shutdown** - Handle SIGTERM
- **Use restart policies** - For reliability

### **Security Best Practices**

- **Scan images for vulnerabilities** - Use Trivy, Snyk
- **Use secrets management** - Don't hardcode secrets
- **Keep images updated** - Regular base image updates
- **Use minimal base images** - Alpine, Distroless
- **Sign images** - Content trust
- **Run read-only filesystem** - When possible
- **Limit capabilities** - Drop unnecessary Linux capabilities

------------------------------------------------------------------------

## Debugging & Troubleshooting

```bash
# View container logs
docker logs mycontainer
docker logs -f mycontainer      # Follow logs
docker logs --tail 100 mycontainer

# Execute command in running container
docker exec -it mycontainer bash
docker exec mycontainer ps aux

# Inspect container
docker inspect mycontainer
docker inspect --format='{{.State}}' mycontainer

# View resource usage
docker stats mycontainer

# View running processes
docker top mycontainer

# Diff (see changes)
docker diff mycontainer

# Copy files from container
docker cp mycontainer:/app/file.txt ./file.txt

# Copy files to container
docker cp ./file.txt mycontainer:/app/file.txt

# View events
docker events

# Common issues:
# - Port already in use: docker run -p 8081:80 ...
# - Container exits immediately: docker logs mycontainer
# - Cannot connect to service: docker network inspect my-network
# - Out of memory: docker run --memory=1g ...
```

------------------------------------------------------------------------

## Docker Compose Overview

Manage multi-container applications:

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db
    volumes:
      - ./app:/app
    networks:
      - my-network
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - my-network
    ports:
      - "5432:5432"

volumes:
  db-data:

networks:
  my-network:
    driver: bridge
```

------------------------------------------------------------------------

## Common Patterns

### **Python Application**

```dockerfile
FROM python:3.11-slim

WORKDIR /app
RUN pip install --upgrade pip

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000
CMD ["python", "app.py"]
```

### **Node.js Application**

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

### **Go Application**

```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM alpine:latest
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

------------------------------------------------------------------------

## Performance Tips

- **Use Alpine images** - Significantly smaller
- **Multi-stage builds** - Reduce final image size
- **Layer caching** - Order Dockerfile for optimal caching
- **Parallel builds** - Build multiple images concurrently
- **Limit context** - Use .dockerignore to exclude files
- **Minimize layers** - Combine RUN commands
- **Use BuildKit** - Faster, more efficient builds
- **Prune regularly** - Clean up unused images/containers/volumes

------------------------------------------------------------------------

## Common Commands Reference

```bash
# Image commands
docker build -t myimage:1.0 .
docker images
docker pull myimage:1.0
docker push myimage:1.0
docker tag myimage:1.0 myimage:latest
docker rmi myimage:1.0

# Container commands
docker run -d myimage:1.0
docker ps -a
docker logs mycontainer
docker exec -it mycontainer bash
docker stop mycontainer
docker start mycontainer
docker rm mycontainer

# Network commands
docker network create my-network
docker network ls
docker network inspect my-network

# Volume commands
docker volume create my-volume
docker volume ls
docker volume inspect my-volume

# System commands
docker system prune       # Clean up unused resources
docker stats              # View resource usage
docker version            # Show Docker version
docker info              # Show Docker info
```

------------------------------------------------------------------------

## Troubleshooting Quick Guide

| Problem | Solution |
|---------|----------|
| Container exits immediately | Check logs: `docker logs mycontainer` |
| Port already in use | Use different port: `docker run -p 8081:80` |
| Cannot connect to service | Check network: `docker network inspect my-network` |
| Out of memory | Increase limit: `docker run --memory=2g` |
| Slow builds | Use BuildKit: `DOCKER_BUILDKIT=1 docker build` |
| Image too large | Use multi-stage build, Alpine base |
| Permission denied | Run with sudo or add user to docker group |
| Volume not mounting | Check path permissions and syntax |
