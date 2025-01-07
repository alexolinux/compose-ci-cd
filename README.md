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

* `.env` file:
  
  `.env` example content (see `env_template`):
  
  ```shell
  WORKSPACE="${HOME}/workspace"
  ARCH="amd64"
  TZ="UTC"
  HOST_UID=1001
  HOST_GID=1001
  JENKINS_PORT=8080
  JENKINS_AGENT_PORT=50000
  CODE_VERSION="4.95.2-39"
  CODE_PORT=8081
  ```

  Load `.env`

  ```shell
  source .env
  ```

* Jenkins Docker Volume:

  ```shell
  docker volume create jenkins_home`
  ```

**Author:**
Alex Mendes

<https://www.linkedin.com/in/mendesalex/>

