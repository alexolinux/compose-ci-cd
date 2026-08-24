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
    tar -xzf "helm-${HELM_VERSION}-linux-${H_AR}.tar.gz" && mv linux-${H_AR}/helm . && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-${A_AR}-${AWS_CLI_VERSION}.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && rm awscliv2.zip

# Stage 2: Final Service Stage
ARG CODE_VERSION
FROM codercom/code-server:${CODE_VERSION:-bookworm}

ARG HOST_UID=1000
ARG HOST_GID=1000
ARG CONTAINER_USER=coder

ENV HOME=/home/${CONTAINER_USER} \
    SHELL=/bin/zsh

USER root

# Detect package manager and install dependencies dynamically
RUN if command -v apt-get >/dev/null; then \
      apt-get update && \
      apt-get install -y --no-install-recommends git zsh vim sudo procps util-linux && \
      apt-get clean && rm -rf /var/lib/apt/lists/*; \
    else \
      dnf update -y && \
      dnf install -y git zsh vim-enhanced util-linux procps-ng shadow-utils sudo && \
      dnf clean all && rm -rf /var/cache/dnf; \
    fi

# Add sudo support for CONTAINER_USER
RUN echo "${CONTAINER_USER} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${CONTAINER_USER} && \
    chmod 0440 /etc/sudoers.d/${CONTAINER_USER}

# Rename user and group to match CONTAINER_USER and IDs
RUN if [ "${CONTAINER_USER}" != "coder" ]; then \
      usermod -l ${CONTAINER_USER} -d /home/${CONTAINER_USER} -m coder && \
      groupmod -n ${CONTAINER_USER} coder; \
    fi && \
    usermod -u ${HOST_UID} ${CONTAINER_USER} && \
    groupmod -g ${HOST_GID} ${CONTAINER_USER} || true

# Update fixuid configuration to match CONTAINER_USER
RUN if [ -f /etc/fixuid/config.yml ]; then \
      sed -i "s/user: coder/user: ${CONTAINER_USER}/g" /etc/fixuid/config.yml && \
      sed -i "s/group: coder/group: ${CONTAINER_USER}/g" /etc/fixuid/config.yml && \
      sed -i "s|/home/coder|/home/${CONTAINER_USER}|g" /etc/fixuid/config.yml; \
    fi

# Copy binaries from shared tools-downloader
COPY --from=tools-downloader /downloads/terraform /usr/local/bin/
COPY --from=tools-downloader /downloads/kubectl /usr/local/bin/
COPY --from=tools-downloader /downloads/helm /usr/local/bin/
COPY --from=tools-downloader /downloads/aws /tmp/aws
RUN /tmp/aws/install && rm -rf /tmp/aws

# Personal configuration, directories and ZSH Enhancements
RUN mkdir -p /home/${CONTAINER_USER}/.config/code-server /home/${CONTAINER_USER}/workspace /home/${CONTAINER_USER}/.local && \
    touch /home/${CONTAINER_USER}/.zshrc && \
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
set showcmd
set wildmenu
set hlsearch
set incsearch
set nu
set ai
set si
set et
set ts=2
set sw=2
set cursorline
set cursorcolumn
set background=dark
colorscheme industry
EOF

# Default command for code-server
CMD ["code-server", "--bind-addr", "0.0.0.0:8080", "."]

