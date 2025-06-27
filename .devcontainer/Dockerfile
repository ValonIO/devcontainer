# Use the official Ubuntu base image
FROM ubuntu:latest

# Install basic utilities and dependencies
RUN apt-get update && \
    apt-get install -y \
    sudo \
    curl \
    gnupg \
    apt-transport-https \
    git \
    software-properties-common && \
    rm -rf /var/lib/apt/lists/*

# Set ARG for non-root user
ARG USER=coder

# Create a non-root user and configure sudo
RUN useradd --create-home --shell /bin/bash ${USER} && \
    echo "${USER} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${USER} && \
    chmod 0440 /etc/sudoers.d/${USER}

# Install Golang
RUN apt-get update && \
    apt-get install -y golang-go && \
    rm -rf /var/lib/apt/lists/*

# Set environment variables
ENV GOPATH=/home/${USER}/go
ENV PATH=/usr/local/go/bin:$GOPATH/bin:$PATH

# Create directories and adjust permissions before switching user
RUN mkdir -p ${GOPATH}/src ${GOPATH}/bin && \
    chown -R ${USER}:${USER} ${GOPATH} && \
    chown -R ${USER}:${USER} /home/${USER}

# Install Docker CLI
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && \
    sudo add-apt-repository \
    "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable" && \
    apt-get update && \
    apt-get install -y docker-ce-cli && \
    rm -rf /var/lib/apt/lists/*

# Install Kind via Go
RUN go install sigs.k8s.io/kind@v0.27.0

# Validate Go installation (optional debugging step)
RUN go version

# Install Kubectl
RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

# Add Helm APT repository and install Helm
RUN curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list && \
    apt-get update && \
    apt-get install -y helm

# Switch to the non-root user
USER ${USER}

# Set the working directory
WORKDIR /home/${USER}

# Ensure Go directories within user environment
RUN mkdir -p ${GOPATH}/src ${GOPATH}/bin

# Add a default command to keep the container running
CMD ["tail", "-f", "/dev/null"]
