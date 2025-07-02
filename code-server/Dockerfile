ARG CODE_VERSION=4.101.2-39
#https://hub.docker.com/r/codercom/code-server/tags
FROM codercom/code-server:${CODE_VERSION}

ARG ARCH
ARG HOST_UID=1000
ARG HOST_GID=1000

# Terraform Version Release - https://releases.hashicorp.com/terraform/
ENV TF_VER=1.12.0

USER root

RUN dnf update -y && \
    dnf install -y \
    curl unzip gnupg2 ca-certificates openssl vim-enhanced util-linux zsh && \
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

RUN mkdir -p $HOME/.local/bin && \
    mkdir $HOME/workspace && \
    touch $HOME/.zshrc && \
    export PATH=$PATH:$HOME/.local/bin && \
    git clone https://github.com/alexolinux/zsh-empowered-productivity.git /tmp/zsh-empowered-productivity && \
    /bin/bash /tmp/zsh-empowered-productivity/install.sh && \
    git clone https://github.com/alexolinux/zsh-kube-enhancer.git /tmp/zsh-kube-enhancer && \
    /bin/bash /tmp/zsh-kube-enhancer/install.sh && \
    sudo rm -rf /tmp/zsh-* && \
    sudo chsh -s $(command -v zsh) && \
    exec zsh 

# Set the working directory
WORKDIR $HOME/workspace
