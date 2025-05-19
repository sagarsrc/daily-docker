#####
# This Dockerfile sets up a CUDA 12.8.1 development environment on Ubuntu 22.04.
# It features an SSH server for remote access, essential development tools,
# the uv package manager for Python dependency management, and a pre-configured Python virtual environment.
#####

FROM nvidia/cuda:12.8.1-devel-ubuntu22.04

# Install SSH server and development utilities
RUN apt-get update && \
    apt-get install -y openssh-server vim nano iputils-ping curl wget git build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install uv
COPY --from=ghcr.io/astral-sh/uv:0.6.9 /uv /uvx /bin/

# Set virtual environment location
ENV VENV_PATH=/root/.venv

# Create Python virtual environment
RUN uv venv ${VENV_PATH} --python=3.12
# Add venv to PATH
ENV PATH=${VENV_PATH}/bin:$PATH
# Install packages using uv with the venv
RUN uv pip install --upgrade pip
RUN uv pip install pandas numpy matplotlib seaborn scikit-learn scipy jupyter tqdm

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