#####
# This Dockerfile configures a CUDA 12.8.1 development environment using Ubuntu 22.04 as its base image.
# It includes an SSH server for remote access, a suite of essential development tools,
# the uv package manager for efficient Python dependency management, and Docker-in-Docker support
# for building and running containers within the container.
#####

FROM nvidia/cuda:12.8.1-devel-ubuntu22.04

# Install SSH server, development utilities, and Docker dependencies
RUN apt-get update && \
    apt-get install -y \
    openssh-server vim nano iputils-ping curl wget git build-essential \
    ca-certificates gnupg lsb-release iptables supervisor && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add Docker's official GPG key and repository
RUN install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    chmod a+r /etc/apt/keyrings/docker.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine and Docker Compose
RUN apt-get update && \
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install uv
COPY --from=ghcr.io/astral-sh/uv:0.6.9 /uv /uvx /bin/

# Configure SSH
RUN mkdir -p /var/run/sshd
RUN echo 'root:yourpassword' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config

# SSH login fix
RUN sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

# Set environment variables for NVIDIA GPU support
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility

# Add CUDA and Docker to PATH for SSH sessions
RUN echo "export PATH=$PATH" >> /etc/profile && \
    echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH" >> /etc/profile && \
    echo "ldconfig" >> /etc/profile

# Create supervisor configuration for running both SSH and Docker daemon
RUN mkdir -p /var/log/supervisor
COPY --from=scratch /dev/null /etc/supervisor/conf.d/supervisord.conf
RUN echo '[supervisord]' > /etc/supervisor/conf.d/supervisord.conf && \
    echo 'nodaemon=true' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo '' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo '[program:sshd]' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'command=/usr/sbin/sshd -D' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'autostart=true' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'autorestart=true' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'stdout_logfile=/var/log/supervisor/sshd.log' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'stderr_logfile=/var/log/supervisor/sshd_err.log' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo '' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo '[program:dockerd]' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'command=/usr/bin/dockerd --host=unix:///var/run/docker.sock --host=tcp://0.0.0.0:2375' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'autostart=true' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'autorestart=true' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'stdout_logfile=/var/log/supervisor/dockerd.log' >> /etc/supervisor/conf.d/supervisord.conf && \
    echo 'stderr_logfile=/var/log/supervisor/dockerd_err.log' >> /etc/supervisor/conf.d/supervisord.conf

# Create Docker group and add root to it
RUN groupadd -f docker && usermod -aG docker root

# Expose SSH port and Docker daemon port
EXPOSE 22 2375

# Use supervisor to run both SSH and Docker daemon
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]