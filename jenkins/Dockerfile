# Import shared tools downloader stage
FROM Dockerfile.tools AS tools-downloader

# Final Stage
FROM jenkins/jenkins

ARG JENKINS_VERSION

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