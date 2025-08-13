# NVIDIA CUDA with Docker-in-Docker Support

This Docker image provides a CUDA 12.8.1 development environment with SSH access and Docker-in-Docker (DinD) support, allowing you to build and run Docker containers within the container.

## Features

- **CUDA 12.8.1** development environment
- **Ubuntu 22.04** base
- **SSH Server** for remote access
- **Docker Engine** and **Docker Compose** pre-installed
- **Docker-in-Docker** support for building containers
- **uv Package Manager** for Python dependencies
- **Development Tools**: git, vim, nano, build-essential, etc.

## Build Instructions

```bash
# 1. Log in to Docker Hub
sudo docker login

# 2. Build your CUDA image with Docker support
sudo docker build -t sagx/nvidia-cuda-ai-dev-docker:latest -f Dockerfile .

# 3. Push to Docker Hub
sudo docker push sagx/nvidia-cuda-ai-dev-docker:latest
```

## Running the Container

To run this container with full Docker-in-Docker support:

```bash
# Run with privileged mode (required for Docker-in-Docker)
docker run -d \
  --name cuda-docker-dev \
  --privileged \
  --gpus all \
  -p 2222:22 \
  -p 2375:2375 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  sagx/nvidia-cuda-ai-dev-docker:latest

# SSH into the container
ssh -p 2222 root@localhost
# Default password: yourpassword
```

## Docker Usage Inside Container

Once inside the container, you can use Docker commands:

```bash
# Check Docker is running
docker version

# Build an image
docker build -t myapp .

# Run containers
docker run hello-world

# Use Docker Compose
docker compose up
```

## Security Note

- The default root password is `yourpassword` - **change this in production**
- Running with `--privileged` flag grants elevated permissions
- Docker daemon is exposed on port 2375 (TCP) - secure this in production
- This setup is intended for development environments only

## GPU Access

The container is configured with full NVIDIA GPU support. CUDA paths are properly configured for both direct use and SSH sessions.