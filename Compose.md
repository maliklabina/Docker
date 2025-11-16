# Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications using a YAML configuration file.

------------------------------------------------------------------------

## Overview

Docker Compose simplifies the process of running multiple interconnected containers with a single command, managing networking, volumes, and environment variables automatically.

### **Key Benefits**

- **Single command deployment** - Run entire application stack
- **Automatic networking** - Containers can communicate by service name
- **Environment management** - Define all config in one file
- **Volume management** - Persistent data across containers
- **Service orchestration** - Manage dependencies and startup order
- **Development workflow** - Same setup across all environments

------------------------------------------------------------------------

## Installation

```bash
# Linux
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# macOS (via Homebrew)
brew install docker-compose

# Verify installation
docker-compose --version
```

------------------------------------------------------------------------

## Basic Compose File Structure

```yaml
version: '3.8'  # Compose file format version

services:       # Define containers
  service-name:
    image: image-name:tag
    ports:
      - "8080:80"
    environment:
      - VAR=value
    volumes:
      - ./local:/container
    networks:
      - my-network

volumes:        # Define volumes
  my-volume:

networks:       # Define networks
  my-network:
    driver: bridge
```

------------------------------------------------------------------------

## Services Configuration

### **Using Images**

```yaml
services:
  web:
    image: nginx:latest
```

### **Building from Dockerfile**

```yaml
services:
  web:
    build: .
    # or
    build:
      context: .
      dockerfile: Dockerfile.prod
      args:
        NODE_ENV: production
```

### **Ports**

```yaml
services:
  web:
    ports:
      - "8080:80"           # Port mapping
      - "8443:443/tcp"      # TCP protocol
      - "8888:8888/udp"     # UDP protocol
      - "127.0.0.1:8080:80" # Specific IP
```

### **Environment Variables**

```yaml
services:
  db:
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_USER=admin
      - POSTGRES_DB=mydb

    # or using env_file
    env_file:
      - .env
      - .env.prod
```

### **Volumes**

```yaml
services:
  web:
    volumes:
      - ./app:/app                 # Bind mount
      - app-data:/data             # Named volume
      - /tmp:/tmp:ro               # Read-only
      - /var/run/docker.sock:/var/run/docker.sock  # Docker socket
```

### **Networks**

```yaml
services:
  web:
    networks:
      - frontend
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

------------------------------------------------------------------------

## Best Practices

- **Version 3.8+** - Use latest stable version
- **Health checks** - Define service health checks
- **Depends_on** - Manage service startup order
- **Resource limits** - Set memory and CPU limits
- **Restart policies** - Use unless-stopped for production
- **Environment files** - Use .env for secrets
- **Volumes for data** - Persistent storage with named volumes
- **Network isolation** - Separate frontend and backend networks

------------------------------------------------------------------------

## Quick Reference Commands

```bash
# Daily commands
docker-compose up -d              # Start services
docker-compose down               # Stop services
docker-compose logs -f            # View logs
docker-compose ps                 # Check status
docker-compose exec api bash      # Access service
docker-compose build              # Build images
docker-compose pull               # Update images
docker-compose restart api        # Restart service
docker-compose rm                 # Remove containers
docker-compose down -v            # Clean everything
```

------------------------------------------------------------------------

## Docker Compose Commands

```bash
# Start services in background
docker-compose up -d

# Start services with build
docker-compose up -d --build

# View logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f api

# Check service status
docker-compose ps

# Execute command in service
docker-compose exec api bash

# Stop services
docker-compose stop

# Stop and remove containers
docker-compose down

# Remove everything including volumes
docker-compose down -v

# Build images
docker-compose build

# Restart service
docker-compose restart api

# Pull latest images
docker-compose pull

# Run command in new container
docker-compose run web npm test

# Validate compose file
docker-compose config

# Remove stopped containers
docker-compose rm
```

------------------------------------------------------------------------

## Environment Management

### **.env File**

```bash
# .env file
DB_HOST=db
DB_USER=admin
DB_PASSWORD=secret
DB_NAME=myapp
REDIS_URL=redis://cache:6379
NODE_ENV=production
API_PORT=5000
```

### **Using .env in Compose**

```yaml
services:
  api:
    environment:
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:5432/${DB_NAME}
      - REDIS_URL=${REDIS_URL}
      - NODE_ENV=${NODE_ENV}
```

### **Override Environment**

```bash
# Use different env file
docker-compose --env-file .env.prod up

# Override specific variable
docker-compose -e NODE_ENV=staging up
```

------------------------------------------------------------------------

## Networking

### **Service Discovery**

Services can communicate using service names:

```bash
# From web service, connect to api
curl http://api:5000

# From api, connect to db
psql -h db -U user -d myapp
```

### **Network Types**

```yaml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

### **External Networks**

```yaml
networks:
  existing-network:
    external: true
```

------------------------------------------------------------------------

## Volumes

### **Named Volumes**

```yaml
volumes:
  db-data:
  cache-data:

services:
  db:
    volumes:
      - db-data:/var/lib/postgresql/data
```

### **Bind Mounts**

```yaml
services:
  web:
    volumes:
      - ./src:/app/src      # Current directory
      - /host/path:/app     # Absolute path
```

### **Volume Options**

```yaml
volumes:
  db-data:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
```

------------------------------------------------------------------------


## Quick Reference Commands

```bash
# Daily commands
docker-compose up -d              # Start services
docker-compose down               # Stop services
docker-compose logs -f            # View logs
docker-compose ps                 # Check status
docker-compose exec api bash      # Access service
docker-compose build              # Build images
docker-compose pull               # Update images
docker-compose restart api        # Restart service
docker-compose rm                 # Remove containers
docker-compose down -v            # Clean everything
```
