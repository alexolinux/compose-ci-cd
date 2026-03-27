# Stage 1: Shared multi-arch tools downloader
FROM alpine:3.19 AS tools-downloader

ARG ARCH
ARG TF_VERSION
ARG KUBECTL_VERSION
ARG HELM_VERSION
ARG AWS_CLI_VERSION

RUN apk add --no-cache curl unzip tar

WORKDIR /downloads

# Download Tools
RUN if [ "${ARCH}" = "amd64" ]; then AR="amd64"; K_AR="amd64"; H_AR="amd64"; A_AR="x86_64"; else AR="arm64"; K_AR="arm64"; H_AR="arm64"; A_AR="aarch64"; fi && \
    curl -LO "https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_${AR}.zip" && \
    unzip "terraform_${TF_VERSION}_linux_${AR}.zip" && rm "terraform_${TF_VERSION}_linux_${AR}.zip" && \
    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${K_AR}/kubectl" && chmod +x kubectl && \
    curl -LO "https://get.helm.sh/helm-${HELM_VERSION}-linux-${H_AR}.tar.gz" && \
    tar -xzf "helm-${HELM_VERSION}-linux-${H_ARCH:-$H_AR}.tar.gz" && mv linux-${H_ARCH:-$H_AR}/helm . && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-${A_AR}-${AWS_CLI_VERSION}.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && rm awscliv2.zip

# Stage 2: Final Jenkins Stage
ARG JENKINS_VERSION
FROM jenkins/jenkins:${JENKINS_VERSION:-latest}

USER root

# Install necessary packages and Docker CLI
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg2 \
        lsb-release \
        unzip \
        jq && \
    mkdir -p /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends docker-ce-cli && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy binaries from shared tools-downloader
COPY --from=tools-downloader /downloads/terraform /usr/local/bin/
COPY --from=tools-downloader /downloads/kubectl /usr/local/bin/
COPY --from=tools-downloader /downloads/helm /usr/local/bin/
COPY --from=tools-downloader /downloads/aws /tmp/aws
RUN /tmp/aws/install && rm -rf /tmp/aws

# Add jenkins user to docker group (optional but helpful if GID matches)
# RUN groupadd -g 999 docker && usermod -aG docker jenkins

# Switch back to Jenkins user
USER jenkins