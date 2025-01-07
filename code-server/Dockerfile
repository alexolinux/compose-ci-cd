ARG CODE_VERSION=4.96.2-39
FROM codercom/code-server:${CODE_VERSION}

ARG ARCH
ARG HOST_UID=1000
ARG HOST_GID=1000

# Terraform Version Release - https://releases.hashicorp.com/terraform/
ENV TF_VER=1.10.3

USER root

RUN dnf update -y && \
    dnf install -y \
    curl unzip gnupg2 ca-certificates openssl && \
    dnf clean all

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-$(arch).zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf aws awscliv2.zip && \
    rm -rf ./aws

RUN curl -LO "https://releases.hashicorp.com/terraform/${TF_VER}/terraform_${TF_VER}_linux_${ARCH}.zip" && \
    unzip terraform_${TF_VER}_linux_${ARCH}.zip && \
    mv terraform /usr/local/bin && \
    rm -f terraform_*.zip

RUN curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/${ARCH}/kubectl" && \
    chmod +x ./kubectl && \
    mv ./kubectl /usr/local/bin/kubectl

RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | sh

ENV PATH="/usr/local/bin:${PATH}"

USER ${HOST_UID}:${HOST_GID}

# Set the working directory
WORKDIR /home/coder/project

