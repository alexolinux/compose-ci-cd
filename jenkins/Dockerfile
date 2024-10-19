FROM jenkins/jenkins:latest

# Terraform Version Release - https://releases.hashicorp.com/terraform/
ENV TF_VER=1.9.0
ARG ARCH

# Switch to root user
USER root

# Install necessary packages
RUN apt-get update && \
    apt-get install -y apt-transport-https \
                       ca-certificates \
                       curl \
                       gnupg2 \
                       software-properties-common \
                       lsb-release \
                       unzip \
                       jq

# Install Docker CE CLI (Check the SO Architecture)
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - && \
    add-apt-repository "deb [arch=${ARCH}] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
    apt-get update && \
    apt-get install -y docker-ce-cli

# Install kubectl
RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/${ARCH}/kubectl" && \
    chmod +x ./kubectl && \
    mv ./kubectl /usr/local/bin/kubectl

# Install Helm
RUN curl -fsSL -o /tmp/get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 && \
    chmod +x /tmp/get_helm.sh && \
    /tmp/get_helm.sh && \
    rm -f /tmp/get_helm.sh

# Install terraform
RUN curl -LO "https://releases.hashicorp.com/terraform/${TF_VER}/terraform_${TF_VER}_linux_${ARCH}.zip" && \
    unzip terraform_${TF_VER}_linux_${ARCH}.zip && \
    mv terraform /usr/local/bin && \
    rm -f terraform_*.zip

# Install AWS CLI v2
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-$(arch).zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm awscliv2.zip && \
    rm -rf ./aws

# Add /usr/local/bin to PATH
ENV PATH="/usr/local/bin:${PATH}"

# Switch back to Jenkins user
USER jenkins

