ARG CODE_VERSION=4.112.0-39

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
FROM codercom/code-server:${CODE_VERSION}

ARG HOST_UID=1000
ARG HOST_GID=1000
ARG CONTAINER_USER=coder

USER root

# Install runtime dependencies and clean up
RUN dnf update -y && \
    dnf install -y git zsh vim-enhanced util-linux procps-ng shadow-utils && \
    dnf clean all && \
    rm -rf /var/cache/dnf

# Rename user and group to match CONTAINER_USER and IDs
RUN if [ "${CONTAINER_USER}" != "coder" ]; then \
      usermod -l ${CONTAINER_USER} -d /home/${CONTAINER_USER} -m coder && \
      groupmod -n ${CONTAINER_USER} coder; \
    fi && \
    usermod -u ${HOST_UID} ${CONTAINER_USER} && \
    groupmod -g ${HOST_GID} ${CONTAINER_USER}

# Copy binaries from downloader
COPY --from=downloader /downloads/terraform /usr/local/bin/
COPY --from=downloader /downloads/kubectl /usr/local/bin/
COPY --from=downloader /downloads/helm /usr/local/bin/
COPY --from=downloader /downloads/aws /tmp/aws
RUN /tmp/aws/install && rm -rf /tmp/aws

# Environment and Shell setup
ENV PATH="/usr/local/bin:${PATH}"

USER root

# Personal configuration and ZSH Enhancements
RUN touch /home/${CONTAINER_USER}/.zshrc && \
    git clone --depth 1 https://github.com/alexolinux/zsh-empowered-productivity.git /tmp/zsh-p && \
    USER=${CONTAINER_USER} HOME=/home/${CONTAINER_USER} /bin/bash /tmp/zsh-p/install.sh && \
    git clone --depth 1 https://github.com/alexolinux/zsh-kube-enhancer.git /tmp/zsh-k && \
    USER=${CONTAINER_USER} HOME=/home/${CONTAINER_USER} /bin/bash /tmp/zsh-k/install.sh && \
    rm -rf /tmp/zsh-* && \
    chsh -s $(command -v zsh) ${CONTAINER_USER} && \
    chown -R ${HOST_UID}:${HOST_GID} /home/${CONTAINER_USER}

USER ${HOST_UID}:${HOST_GID}
WORKDIR /home/${CONTAINER_USER}/workspace
SHELL ["/bin/zsh", "-c"]

# SSH and Git Config
RUN mkdir -p $HOME/.ssh && chmod 700 $HOME/.ssh && \
    git config --global init.defaultBranch main && \
    git config --global pull.rebase false

RUN cat <<EOF > $HOME/.vimrc
syntax on
set ai
set si
set et
set hlsearch
set wildmenu
set showcmd
set ts=2
set sts=2
set sw=2
EOF

WORKDIR /home/${CONTAINER_USER}/workspace

# Default command for code-server
CMD ["code-server", "--bind-addr", "0.0.0.0:8080", "."]

