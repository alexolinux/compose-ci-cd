# compose-ci-cd

---------------

Orchestrated CI/CD for Personal Lab (docker-compose)

Structure

```shell
.
├── code-server
├── docker-compose.yml
├── jenkins
│   └── Dockerfile
├── LICENSE
├── .env
└── README.md
```

## Requirements

* `.env` file:
  
  `.env` example content:
  
  ```shell
  WORKSPACE="${HOME}/workspace"
  ARCH="amd64"
  TZ="UTC"
  UID=1001
  GID=1001
  JENKINS_PORT=8080
  JENKINS_AGENT_PORT=50000
  CODE_PORT=8081
  ```

* Jenkins Docker Volume:

  ```shell
  docker volume create jenkins_home`
  ```

Author Information
Alex Mendes

<https://www.linkedin.com/in/mendesalex/>
