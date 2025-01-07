# Declare build arguments and provide a default value for CODE_VERSION
ARG CODE_VERSION=4.96.2-39

# Use the argument in the FROM directive
FROM codercom/code-server:${CODE_VERSION}

# Declare additional build arguments
ARG HOST_UID=1000
ARG HOST_GID=1000

# Switch to root for installations
USER root

# Install dependencies using dnf
RUN dnf update -y && \
    dnf install -y \
    curl unzip && \
    dnf clean all

# Download and install AWS CLI for aarch64
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf aws awscliv2.zip

# Switch back to the specified user
USER ${HOST_UID}:${HOST_GID}

# Set the working directory
WORKDIR /home/coder/project

