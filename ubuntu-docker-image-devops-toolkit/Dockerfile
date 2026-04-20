# Use Ubuntu 24.04 LTS (Noble Numbat) as the base image
FROM ubuntu:24.04

# Set environment variables to avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm

# Update package lists and install base tools
# Clean up apt cache in the same layer to keep image size small
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Essential CLI tools
    ca-certificates \
    curl \
    wget \
    git \
    vim \
    nano \
    unzip \
    zip \
    jq \
    # Build tools (needed for some Python packages or manual builds)
    build-essential \
    # Process and system monitoring
    htop \
    net-tools \
    iputils-ping \
    # Container / orchestration tools
    docker.io \
    # Ansible dependencies (Python3 & pip)
    python3 \
    python3-pip \
    python3-venv \
    # SSH client for git over SSH or remote connections
    openssh-client \
    # Miscellaneous useful utilities
    gnupg \
    lsb-release \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install additional DevOps tools not available via apt
# ----------------------------------------------------
# 1. Install yq (YAML processor) – standalone binary
RUN wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 \
    && chmod +x /usr/local/bin/yq

# 2. Install Docker Compose plugin (standalone)
RUN mkdir -p /usr/local/lib/docker/cli-plugins \
    && curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose \
    && chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

# 3. Install kubectl (Kubernetes CLI)
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/

# 4. Install Terraform (using HashiCorp's official repository for automatic updates)
RUN wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 5. Install Helm (Kubernetes package manager)
RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \
    && chmod +x get_helm.sh \
    && ./get_helm.sh \
    && rm get_helm.sh

# 6. Install Ansible (via pip, because apt version is often older)
RUN pip3 install --no-cache-dir ansible

# 7. Install AWS CLI v2 (for cloud DevOps tasks)
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install \
    && rm -rf awscliv2.zip aws

# Create a non‑root user to follow security best practices
# The user 'devops' will have sudo access (without password) for convenience
ARG USERNAME=devops
ARG USER_UID=1000
ARG USER_GID=1000

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

# Set the working directory to the user's home folder
WORKDIR /home/$USERNAME

# Switch to the non‑root user
USER $USERNAME

# Optionally add custom bash aliases for convenience
RUN echo "alias ll='ls -alF'" >> ~/.bashrc \
    && echo "alias k='kubectl'" >> ~/.bashrc \
    && echo "alias tf='terraform'" >> ~/.bashrc

# Final instructions to the user
CMD ["/bin/bash"]
