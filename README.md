# Compose CI/CD - Personal Lab

---------------

A highly optimized, multi-architecture (AMD64/ARM64) Orchestrated CI/CD Environment for Personal Labs using Docker Compose.

## 🚀 Featured Services

*   **[Code-Server](https://github.com/coder/code-server)**: VS Code in the browser optimized for DevOps.
    *   Pre-installed: AWS CLI, Helm, Kubectl, Terraform.
    *   ZSH customized for DevOps users.
    *   **Host-User Sync**: Runs with your local UID/GID to avoid permission issues.
*   **[Jenkins](https://www.jenkins.io/doc/book/installing/docker/)**: Continuous Integration server.
    *   **Slim Edition**: Optimized for Raspberry Pi/ARM (lower memory footprint).
    *   Shared CI/CD toolset synced with Code-Server.
    *   Docker-out-of-Docker (DooD) support.
*   **[n8n](https://n8n.io/)**: Workflow automation tool (Optional/Included).
*   **Git Services**: Supports Gitea/GitLab integration (see `git-compose.yml`).

## 🛠️ Project Structure

```text
.
├── code-server/         # Code-Server customization (Dockerfile, config)
├── jenkins/             # Jenkins customization (Dockerfile, plugins)
├── docker-compose.yml   # Main orchestration
├── .env                 # Unified environment configuration
├── OPTIMIZATION.md      # Detailed technical optimization report
└── README.md
```

## ⚙️ Configuration

The project uses a unified `.env` file to manage all versions and configurations across all containers.

### 1. Preparation

Create your `.env` file based on the provided template or existing configuration:

```bash
# Set your Host identity (crucial for permissions)
USER_NAME="your_user"
HOST_UID=$(id -u)
HOST_GID=$(id -g)

# Set your Architecture
ARCH="arm64" # or "amd64"
```

### 2. Required Volumes

The services expect some external volumes and local paths. Ensure they exist:

```bash
# Create persistent volumes
docker volume create jenkins_home
docker volume create n8n_home

# Ensure local workspace exists
mkdir -p ~/workspace
```

### 3. Usage

Build and start the environment:

```bash
# Build using the optimized multi-stage cache
docker-compose build

# Start the lab
docker-compose up -d
```

## 💎 Key Features

*   **Multi-Arch Support**: Works natively on Raspberry Pi (ARM64) and standard Servers (AMD64).
*   **Shared Tooling**: Terraform, Kubectl, Helm, and AWS CLI are downloaded once and shared across all CI/CD containers.
*   **Permission Harmony**: No more `sudo` or permission denied errors. The containers adopt your host user's UID/GID.
*   **Optimized Build**: Multi-stage Docker builds reduce image sizes by up to 40%.

---
**Author:**
Alex Mendes - [LinkedIn](https://www.linkedin.com/in/mendesalex/)

