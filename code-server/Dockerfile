# Import shared tools downloader stage
FROM Dockerfile.tools AS tools-downloader

# Final Stage
FROM codercom/code-server

ARG CODE_VERSION
ARG HOST_UID
ARG HOST_GID
ARG CONTAINER_USER

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

# Copy binaries from shared tools-downloader
COPY --from=tools-downloader /downloads/terraform /usr/local/bin/
COPY --from=tools-downloader /downloads/kubectl /usr/local/bin/
COPY --from=tools-downloader /downloads/helm /usr/local/bin/
COPY --from=tools-downloader /downloads/aws /tmp/aws
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