# Dockerfile

A Dockerfile contains instructions to build a Docker image. Each instruction creates a layer in the image.

------------------------------------------------------------------------

## Dockerfile Instructions Reference

### **FROM**
Specifies the base image - must be the first instruction (except for ARG).

```dockerfile
FROM ubuntu:22.04
FROM node:18-alpine
FROM python:3.11-slim
FROM mcr.microsoft.com/dotnet/aspnet:7.0
```

### **RUN**
Executes commands during image build.

```dockerfile
# Single command
RUN apt-get update

# Multiple commands (chain with &&)
RUN apt-get update && apt-get install -y \
    curl \
    git \
    wget

# Shell form
RUN echo "Hello"

# Exec form (preferred)
RUN ["apt-get", "update"]
```

### **WORKDIR**
Sets the working directory for subsequent instructions.

```dockerfile
WORKDIR /app
WORKDIR /src
# Commands run from /app
```

### **COPY**
Copies files from host to container.

```dockerfile
COPY . .
COPY requirements.txt .
COPY ./src /app/src
COPY --chown=user:group file.txt /app/
```

### **ADD**
Like COPY but also handles URLs and tar extraction (less commonly used).

```dockerfile
ADD https://example.com/file.tar.gz .
ADD file.tar.gz /app/
```

### **ENV**
Sets environment variables in the container.

```dockerfile
ENV NODE_ENV=production
ENV DATABASE_URL=postgresql://localhost
ENV PORT=3000
```

### **ARG**
Build-time variables (available during build, not at runtime).

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
ARG BUILD_DATE
```

### **EXPOSE**
Documents which ports the container listens on (doesn't actually publish).

```dockerfile
EXPOSE 3000
EXPOSE 80 443
EXPOSE 5432/tcp
```

### **CMD**
Default command when container starts.

```dockerfile
# Shell form
CMD npm start

# Exec form (preferred)
CMD ["npm", "start"]

# Default parameters for ENTRYPOINT
CMD ["param1", "param2"]
```

### **ENTRYPOINT**
Configures container to run as executable.

```dockerfile
# Shell form
ENTRYPOINT npm start

# Exec form (preferred)
ENTRYPOINT ["npm", "start"]

# Combined with CMD
ENTRYPOINT ["npm"]
CMD ["start"]
# Results in: npm start
```

### **VOLUME**
Defines mount points for volumes.

```dockerfile
VOLUME ["/data", "/logs"]
VOLUME /var/lib/postgresql/data
```

### **USER**
Specifies user to run commands.

```dockerfile
USER root
RUN apt-get install -y curl
USER appuser
RUN some-command
```

### **LABEL**
Adds metadata to image.

```dockerfile
LABEL maintainer="user@example.com"
LABEL version="1.0"
LABEL description="My application"
```

### **HEALTHCHECK**
Defines health check for the container.

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000 || exit 1
```

### **SHELL**
Specifies shell to use (Windows).

```dockerfile
SHELL ["powershell", "-Command"]
```

------------------------------------------------------------------------

## Dockerfile Best Practices

### **Use Specific Base Image Tags**

```dockerfile
# Bad - Latest can change unexpectedly
FROM node:latest

# Good - Specific version
FROM node:18.17.0-alpine3.18
```

### **Minimize Layers - Chain Commands**

```dockerfile
# Bad - Creates multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# Good - Single layer
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

### **Use Alpine Images**

```dockerfile
# Large image (~918 MB)
FROM ubuntu:22.04

# Smaller image (~71 MB)
FROM python:3.11-alpine
```

### **Use Multi-stage Builds**

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
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### **Run as Non-root User**

```dockerfile
# Create user (for Linux images)
RUN useradd -m -u 1000 appuser

# Copy files with correct ownership
COPY --chown=appuser:appuser . /app

# Switch to user
USER appuser

# Commands run as appuser
```


### **Order Instructions for Caching**

```dockerfile
# Base image
FROM node:18-alpine

# Rarely changed
LABEL maintainer="user@example.com"

# Dependencies (cached well)
COPY package*.json ./
RUN npm ci

# Code (changes frequently - put last)
COPY . .

# Build/Run
RUN npm run build
CMD ["npm", "start"]
```

### **Use Health Checks**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm ci
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/ || exit 1

CMD ["npm", "start"]
```

------------------------------------------------------------------------

## Common Dockerfile Examples

### **Node.js Application**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/ || exit 1

USER node

CMD ["node", "server.js"]
```

### **Python Application**

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import requests; requests.get('http://localhost:5000')" || exit 1

CMD ["python", "app.py"]
```

### **Node.js Application with Multi-stage**

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/ || exit 1

USER node

CMD ["node", "dist/index.js"]
```

### **Java Application**

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-alpine

WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### **React Application with Nginx**

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

------------------------------------------------------------------------

## Build Commands

```bash
# Build image
docker build -t myimage:1.0 .

# Build with custom Dockerfile
docker build -f Dockerfile.prod -t myimage:1.0 .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t myimage:1.0 .

# Build with progress
docker build --progress=plain -t myimage:1.0 .

# Build without cache
docker build --no-cache -t myimage:1.0 .

# Build with multiple tags
docker build -t myimage:1.0 -t myimage:latest .

# Build and push
docker build -t registry.azurecr.io/myimage:1.0 . && \
docker push registry.azurecr.io/myimage:1.0
```

------------------------------------------------------------------------

## Optimization Tips

- **Smaller base images** - Alpine, distroless, scratch
- **Multi-stage builds** - Remove build dependencies
- **Combine RUN commands** - Reduce layers
- **Leverage layer caching** - Put changing commands last
- **Minimize context** - Use .dockerignore
- **Remove unnecessary files** - Clean up in same RUN
- **Non-root user** - Better security
- **Health checks** - Container health monitoring

------------------------------------------------------------------------

## Debugging

```bash
# View image layers
docker history myimage:1.0

# View image details
docker inspect myimage:1.0

# Run image interactively
docker run -it myimage:1.0 bash

# Exec into running container
docker exec -it mycontainer bash

# View logs
docker logs mycontainer

# Check build context
docker build --progress=plain -t myimage:1.0 .
```

------------------------------------------------------------------------

## Common Issues

| Issue | Solution |
|-------|----------|
| Image too large | Use multi-stage build, Alpine base |
| Slow builds | Order commands for caching, exclude files |
| Permission denied | Use correct USER, ownership |
| Port not exposed | Add EXPOSE, use -p flag when running |
| Container crashes | Check CMD/ENTRYPOINT, view logs |
| Layer not updating | Use --no-cache, invalidate cache |

------------------------------------------------------------------------

## Quick Reference

```dockerfile
# Minimal Dockerfile
FROM alpine:latest
WORKDIR /app
COPY . .
EXPOSE 8000
CMD ["./app"]

# Production Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
USER node
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget --quiet --tries=1 --spider http://localhost:3000/
CMD ["node", "dist/index.js"]
```
