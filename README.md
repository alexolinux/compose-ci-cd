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
  ```

* Jenkins Docker Volume:

  ```shell
  docker volume create jenkins_home`
  ```

Author Information
Alex Mendes

<https://www.linkedin.com/in/mendesalex/>
