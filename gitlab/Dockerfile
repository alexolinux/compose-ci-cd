# Use an ARM64-compatible OS as base
FROM ubuntu:22.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV GITLAB_HOME=/var/opt/gitlab

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl openssh-server ca-certificates tzdata perl \
    && apt-get clean

# Add GitLab package repository
RUN curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | bash

# Install GitLab CE (ARM64 package)
RUN apt-get update && apt-get install -y gitlab-ce

# Mount GitLab data
VOLUME ["$GITLAB_HOME"]

# Start GitLab services
CMD ["/opt/gitlab/embedded/bin/runsvdir-start"]

