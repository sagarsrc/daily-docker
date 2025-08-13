# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a collection of Docker images optimized for NVIDIA CUDA development environments with SSH access. Each image provides a pre-configured Ubuntu 22.04 environment with CUDA 12.8.1 development tools.

## Available Docker Images

1. **nvidia-cuda-ubuntu-ssh**: Base CUDA development environment with SSH server and uv package manager
2. **nvidia-cuda-ubuntu-ssh-py312**: Extended environment with Python 3.12 virtual environment and pre-installed data science packages (pandas, numpy, matplotlib, seaborn, scikit-learn, scipy, jupyter, tqdm)
3. **nvidia-cuda-ubuntu-ssh-docker**: CUDA environment with Docker-in-Docker support for building and running containers within the container

## Build and Deployment Commands

### Building Images

```bash
# For base CUDA image
cd nvidia-cuda-ubuntu-ssh
sudo docker build -t sagx/nvidia-cuda-ai-dev:latest -f Dockerfile .

# For Python 3.12 enhanced image
cd nvidia-cuda-ubuntu-ssh-py312
sudo docker build -t sagx/nvidia-cuda-ai-dev-py312:latest -f Dockerfile .

# For Docker-in-Docker enabled image
cd nvidia-cuda-ubuntu-ssh-docker
sudo docker build -t sagx/nvidia-cuda-ai-dev-docker:latest -f Dockerfile .
```

### Publishing to Docker Hub

```bash
# Login to Docker Hub
sudo docker login

# Push base image
sudo docker push sagx/nvidia-cuda-ai-dev:latest

# Push Python 3.12 image
sudo docker push sagx/nvidia-cuda-ai-dev-py312:latest

# Push Docker-in-Docker image
sudo docker push sagx/nvidia-cuda-ai-dev-docker:latest
```

## Architecture

Each Docker image follows a similar structure:

1. **Base Layer**: nvidia/cuda:12.8.1-devel-ubuntu22.04 providing CUDA development tools
2. **Development Tools Layer**: SSH server, build-essential, git, vim, nano, and other utilities
3. **Package Management**: uv package manager for Python dependency management
4. **SSH Configuration**: Configured for remote access on port 22 (default password: "yourpassword")
5. **CUDA Environment**: Proper PATH and LD_LIBRARY_PATH configuration for CUDA access over SSH

The Python 3.12 variant adds:
- Pre-configured Python 3.12 virtual environment at `/root/.venv`
- Common data science and ML libraries pre-installed

The Docker-in-Docker variant adds:
- Docker Engine and Docker Compose installed
- Supervisor to manage both SSH and Docker daemons
- Privileged mode support for container builds
- Docker daemon exposed on port 2375

## Key Configuration Details

- SSH root login is enabled (for development environments only)
- NVIDIA GPU support is configured via environment variables
- CUDA paths are properly exported for SSH sessions
- Port 22 is exposed for SSH access