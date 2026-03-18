# Chapter 11: Containers and Virtualization

## Overview
This chapter covers containerization with Docker and basic virtualization concepts essential for modern DevOps and cloud deployment.

## Container Fundamentals

### What are Containers?
- Lightweight, standalone, executable packages
- Include application and dependencies
- Isolated from host and other containers
- Fast and efficient compared to VMs

### Docker Basics
```bash
# Install Docker
sudo apt-get install docker.io

# Docker daemon operations
sudo systemctl start docker
sudo systemctl enable docker

# Check Docker version
docker --version

# Docker info
docker info
```

## Docker Images

### Working with Images
```bash
# Search for images
docker search ubuntu

# Pull an image
docker pull ubuntu:22.04

# List images
docker images

# Remove image
docker rmi image_name

# Build custom image
docker build -t myapp:1.0 .

# Tag image
docker tag myapp:1.0 myregistry/myapp:latest

# Push to registry
docker push myregistry/myapp:latest
```

### Dockerfile
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3
WORKDIR /app
COPY . /app/
RUN pip install -r requirements.txt
EXPOSE 8000
CMD ["python3", "app.py"]
```

## Docker Containers

### Container Management
```bash
# Run container
docker run -d --name mycontainer ubuntu:22.04

# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop container_id

# Start container
docker start container_id

# Remove container
docker rm container_id

# View logs
docker logs container_id

# Execute command in container
docker exec -it container_id bash

# Inspect container
docker inspect container_id
```

## Container Networking

### Network Types
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Connect container to network
docker network connect mynetwork container_id

# Inspect network
docker network inspect mynetwork

# Run container with specific network
docker run -d --network mynetwork --name web nginx
```

## Container Volumes

### Volume Management
```bash
# Create volume
docker volume create myvolume

# List volumes
docker volume ls

# Remove volume
docker volume rm myvolume

# Run container with volume
docker run -d -v myvolume:/data ubuntu

# Run with host directory binding
docker run -d -v /host/path:/container/path ubuntu

# Inspect volume
docker volume inspect myvolume
```

## Docker Compose

### Multi-Container Applications
```yaml
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs

# Stop services
docker-compose down

# Execute command
docker-compose exec service_name command

# Scale service
docker-compose up -d --scale service=3
```

## Container Registry

### Docker Registry
```bash
# Login to registry
docker login

# Push image
docker push username/imagename:tag

# Pull from registry
docker pull username/imagename:tag

# View registry information
docker info
```

## Virtualization Basics

### Virtual Machines vs Containers
- **VMs**: Full OS, larger footprint, more resources
- **Containers**: Share kernel, lightweight, faster startup

### Hypervisors
- **Type 1**: KVM, Xen (runs on hardware)
- **Type 2**: VirtualBox (runs on OS)

### Qemu/KVM
```bash
# Install KVM
sudo apt-get install qemu-kvm libvirt-daemon

# Check KVM support
kvm-ok

# Virtual network operations
virsh net-list

# List virtual machines
virsh list
```

## Practice Exercises

1. **Create Docker Image**:
   - Build custom Dockerfile
   - Create image with application
   - Push to registry

2. **Docker Networking**:
   - Create custom network
   - Run multiple containers
   - Enable container-to-container communication

3. **Docker Compose**:
   - Write docker-compose.yml
   - Run multi-container application
   - Test service dependencies

4. **Volume Management**:
   - Create and manage volumes
   - Implement data persistence
   - Test backup and restore

5. **Container Optimization**:
   - Reduce image size
   - Optimize Dockerfile
   - Monitor container resources

## Key Takeaways
- Containers provide lightweight application deployment
- Docker simplifies container management
- Docker Compose enables multi-container orchestration
- Virtualization and containerization serve different purposes
- Proper image and volume management is crucial
