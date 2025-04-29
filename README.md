# compose-ci-cd

---------------

Orchestrated CI/CD for Personal Lab (docker-compose)

* CI Jenkins with:
  - AWS Cli
  - helm
  - kubectl
  - terraform

* Code-Server with:
  - AWS Cli
  - helm
  - kubectl
  - terraform

Structure

```shell
.
├── code-server
│   └── Dockerfile
├── docker-compose.yml
├── env_template
├── jenkins
│   └── Dockerfile
├── LICENSE
└── README.md
```

## Requirements

* `.env` file: Define the values according to your system configuration.
  
  Create `.env` based on the example content (see `env_template`):

  Load `.env`

  ```shell
  source .env
  ```

:information_source: Changing the Dockerfiles is required:

* code-server
  * `CODE_VERSION`
  * `HOST_UID`
  * `HOST_GID`
  * `TF_VER`

* jenkins
  * `TF_VER`

* gitea
  * `GITEA_VERSION`
  * `GITEA_PORT`
  * `GITEA_SSH_PORT`

### Docker Volumes

* Jenkins Docker Volume:

  ```shell
  docker volume create jenkins_home`
  ```

**Author:**
Alex Mendes

<https://www.linkedin.com/in/mendesalex/>
