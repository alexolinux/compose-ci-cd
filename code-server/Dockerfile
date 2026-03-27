# Import shared tools downloader stage
FROM Dockerfile.tools AS tools-downloader

# Final Stage
ARG CODE_VERSION
FROM codercom/code-server:${CODE_VERSION:-latest}

ARG HOST_UID
ARG HOST_GID
ARG CONTAINER_USER

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

# Copy binaries from shared tools-downloader
COPY --from=tools-downloader /downloads/terraform /usr/local/bin/
COPY --from=tools-downloader /downloads/kubectl /usr/local/bin/
COPY --from=tools-downloader /downloads/helm /usr/local/bin/
COPY --from=tools-downloader /downloads/aws /tmp/aws
RUN /tmp/aws/install && rm -rf /tmp/aws

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

# Default command for code-server
CMD ["code-server", "--bind-addr", "0.0.0.0:8080", "."]
