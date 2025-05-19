#####
# This Dockerfile configures a CUDA 12.8.1 development environment using Ubuntu 22.04 as its base image.
# It includes an SSH server for remote access, a suite of essential development tools,
# and the uv package manager for efficient Python dependency management.
#####

FROM nvidia/cuda:12.8.1-devel-ubuntu22.04

# Install SSH server and development utilities
RUN apt-get update && \
    apt-get install -y openssh-server vim nano iputils-ping curl wget git build-essential && \
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

# Add CUDA to PATH for SSH sessions (critical for CUDA to work over SSH)
RUN echo "export PATH=$PATH" >> /etc/profile && \
    echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH" >> /etc/profile && \
    echo "ldconfig" >> /etc/profile

# Expose SSH port
EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]