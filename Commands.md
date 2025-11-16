# Docker Commands

Essential daily-use Docker commands for efficient container management.

------------------------------------------------------------------------

## Image Commands

```bash
# Build image
docker build -t myimage:1.0 .

# List images
docker images

# Pull image from registry
docker pull ubuntu:22.04

# Push image to registry
docker push myregistry.azurecr.io/myimage:1.0

# Tag image
docker tag myimage:1.0 myregistry.azurecr.io/myimage:1.0

# Remove image
docker rmi myimage:1.0

# View image history
docker history myimage:1.0

# View image details
docker inspect myimage:1.0

# Remove unused images
docker image prune -a
```

------------------------------------------------------------------------

## Container Commands

```bash
# Run container in background
docker run -d --name mycontainer myimage:1.0

# Run with port mapping
docker run -d -p 8080:80 --name mycontainer myimage:1.0

# Run with volume mount
docker run -d -v my-volume:/app/data --name mycontainer myimage:1.0

# Run with environment variables
docker run -d -e DB_HOST=localhost -e DB_PORT=5432 --name mycontainer myimage:1.0

# Run interactive shell
docker run -it myimage:1.0 bash

# List running containers
docker ps

# List all containers
docker ps -a

# View container logs
docker logs mycontainer

# Follow container logs
docker logs -f mycontainer

# View last 100 lines
docker logs --tail 100 mycontainer

# Execute command in container
docker exec -it mycontainer bash

# Stop container
docker stop mycontainer

# Start container
docker start mycontainer

# Restart container
docker restart mycontainer

# Remove container
docker rm mycontainer

# Force remove running container
docker rm -f mycontainer

# View container info
docker inspect mycontainer

# View resource usage
docker stats mycontainer

# Copy file from container
docker cp mycontainer:/app/file.txt ./file.txt

# Copy file to container
docker cp ./file.txt mycontainer:/app/file.txt
```

------------------------------------------------------------------------

## Network Commands

```bash
# Create network
docker network create my-network

# List networks
docker network ls

# Inspect network
docker network inspect my-network

# Connect container to network
docker network connect my-network mycontainer

# Disconnect container from network
docker network disconnect my-network mycontainer

# Remove network
docker network rm my-network
```

------------------------------------------------------------------------

## Volume Commands

```bash
# Create volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove unused volumes
docker volume prune
```

------------------------------------------------------------------------

## System Commands

```bash
# Show Docker version
docker version

# Show Docker info
docker info

# Show system disk usage
docker system df

# Clean up unused resources
docker system prune

# View real-time events
docker events

# View container stats
docker stats
```

------------------------------------------------------------------------

## Quick Daily Workflow

```bash
# Build and run
docker build -t myapp:1.0 .
docker run -d -p 8080:3000 --name myapp myapp:1.0

# Check status
docker ps
docker logs -f myapp

# Stop and remove
docker stop myapp
docker rm myapp

# Clean up
docker system prune
```
