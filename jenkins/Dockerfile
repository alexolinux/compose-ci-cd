ARG JENKINS_VERSION=latest-jdk21

# Multi-arch downloader stage
FROM alpine:3.19 AS downloader

ARG ARCH=amd64
ARG TF_VERSION=1.7.5
ARG KUBECTL_VERSION=v1.29.2
ARG HELM_VERSION=v3.14.2
ARG AWS_CLI_VERSION=2.15.25

RUN apk add --no-cache curl unzip tar

WORKDIR /downloads

# Download Terraform
RUN if [ "${ARCH}" = "amd64" ]; then TF_ARCH="amd64"; else TF_ARCH="arm64"; fi && \
    curl -LO "https://releases.hashicorp.com/terraform/${TF_VERSION}/terraform_${TF_VERSION}_linux_${TF_ARCH}.zip" && \
    unzip "terraform_${TF_VERSION}_linux_${TF_ARCH}.zip" && \
    rm "terraform_${TF_VERSION}_linux_${TF_ARCH}.zip"

# Download Kubectl
RUN if [ "${ARCH}" = "amd64" ]; then K_ARCH="amd64"; else K_ARCH="arm64"; fi && \
    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${K_ARCH}/kubectl" && \
    chmod +x kubectl

# Download Helm
RUN if [ "${ARCH}" = "amd64" ]; then H_ARCH="amd64"; else H_ARCH="arm64"; fi && \
    curl -LO "https://get.helm.sh/helm-${HELM_VERSION}-linux-${H_ARCH}.tar.gz" && \
    tar -xzf "helm-${HELM_VERSION}-linux-${H_ARCH}.tar.gz" && \
    mv linux-${H_ARCH}/helm . && \
    rm -rf linux-${H_ARCH} "helm-${HELM_VERSION}-linux-${H_ARCH}.tar.gz"

# Download AWS CLI
RUN if [ "${ARCH}" = "amd64" ]; then AWS_ARCH="x86_64"; else AWS_ARCH="aarch64"; fi && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}-${AWS_CLI_VERSION}.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    rm awscliv2.zip

# Final Stage
FROM jenkins/jenkins:${JENKINS_VERSION}

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

# Copy binaries from downloader
COPY --from=downloader /downloads/terraform /usr/local/bin/
COPY --from=downloader /downloads/kubectl /usr/local/bin/
COPY --from=downloader /downloads/helm /usr/local/bin/
COPY --from=downloader /downloads/aws /tmp/aws
RUN /tmp/aws/install && rm -rf /tmp/aws

# Add jenkins user to docker group (optional but helpful if GID matches)
# RUN groupadd -g 999 docker && usermod -aG docker jenkins

# Switch back to Jenkins user
USER jenkins
