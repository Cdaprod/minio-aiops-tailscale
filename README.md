# Build with GitHub Actions: MinIO Weaviate Python

#### Workflow Status Badges:
[![Deploy](https://github.com/Cdaprod/minio-aiops-tailscale/actions/workflows/docker-compose-deploy-test.yml/badge.svg)](https://github.com/Cdaprod/minio-aiops-tailscale/actions/workflows/docker-compose-deploy-test.yml)

[![Docker-Compose Build Test](https://github.com/Cdaprod/minio-weaviate/actions/workflows/docker-compose-build-test.yml/badge.svg)](https://github.com/Cdaprod/minio-weaviate/actions/workflows/docker-compose-build-test.yml)

[![Update README with Directory Tree](https://github.com/Cdaprod/minio-weaviate-python/actions/workflows/update_readme.yml/badge.svg)](https://github.com/Cdaprod/minio-weaviate-python/actions/workflows/update_readme.yml)

## Current Directory Tree Structure
The following directory tree is programatically generated to provide an overview of the repos structure (by using `.github/workflows/update_readme.yml` and `.github/scripts/update_readme.py` and is ran on `push` to `main`):


<!-- DIRECTORY_TREE_START -->
```
.
├── DIRECTORY_TREE.txt
├── README.md
├── app
│   ├── Dockerfile
│   ├── entrypoint.sh
│   ├── logs
│   │   └── python_logs.txt
│   ├── main.py
│   └── requirements.txt
├── docker-compose-incorrect_ts_2.yaml
├── docker-compose-incorrect_ts_3.yaml
├── docker-compose.incorrect_ts.yaml
├── docker-compose.minio-weaviate-python.yaml
├── docker-compose.yaml
├── minio
│   ├── Dockerfile
│   ├── entrypoint.sh
│   └── logs
│       └── minio_logs.txt
├── my-docs
│   ├── BLOG_DRAFT.md
│   └── GPT_REFINE_COMPOSE.md
├── tailscale
│   ├── config
│   │   ├── aiops.json
│   │   ├── minio.json
│   │   └── weaviate.json
│   └── docker-compose.tailscale.yaml
└── weaviate
    ├── data.json
    ├── logs
    │   └── weaviate_logs.txt
    └── schema.json

9 directories, 24 files

```
<!-- DIRECTORY_TREE_END -->

## Introduction
This project serves as a template for building [specific type of projects] with a pre-configured setup including `/app/`, `/minio/`, and `/weaviate/` directories.

## How to Use This Template
To start a new project based on this template:

1. Click the "Use this template" button on the GitHub repository page.
2. Choose a name for your new repository and select "Create repository from template".
3. Clone your new repository to your local machine and begin customizing.

## Configuration
- **App**: Instructions on configuring the `/app/` directory.
- **Minio**: Steps to set up the `/minio/` directory.
- **Weaviate**: Guidelines for initializing the `/weaviate/` component.

Based on the available information from the MinIO and Weaviate documentation, here are the specific commands and configurations for setting up MinIO and Weaviate using Dockerfile and `entrypoint.sh`. However, the exact details for MinIO were not found in the recent search, so I'll provide a general approach based on common practices.

### MinIO Setup using Dockerfile and entrypoint.sh

To configure a MinIO server using Docker, you typically start by creating a `Dockerfile` that specifies the MinIO server image and any necessary environment variables. The `entrypoint.sh` script is used to customize the startup behavior of the MinIO server.

#### Dockerfile for MinIO

```Dockerfile
FROM minio/minio
COPY entrypoint.sh /usr/bin/entrypoint.sh
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["/usr/bin/entrypoint.sh"]
```

#### entrypoint.sh for MinIO

```bash
#!/bin/sh
# Custom startup script for MinIO

# Initialize MinIO server with specified directories or buckets
minio server /data --console-address ":9001"
```

You may need to adjust the `minio server` command with additional flags or environment variables as per your configuration requirements, such as enabling TLS, setting access and secret keys, etc.

### Weaviate Setup using Dockerfile and entrypoint.sh

The Weaviate documentation provides insights into running Weaviate with Docker, including using `docker-compose` for orchestrating multiple services. For a single-node setup or development purposes, you can encapsulate the configuration in a Dockerfile and an entrypoint script.

#### Dockerfile for Weaviate

```Dockerfile
FROM semitechnologies/weaviate
COPY entrypoint.sh /usr/bin/entrypoint.sh
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["/usr/bin/entrypoint.sh"]
```

#### entrypoint.sh for Weaviate

```bash
#!/bin/bash
# Custom startup script for Weaviate

# Start Weaviate with a specific configuration
# You can customize this command based on your setup requirements
weaviate start -d
```

In practice, you'd replace `weaviate start -d` with the actual command to start Weaviate, configuring it with environment variables or command-line options as needed for your application. Since Weaviate's setup can vary significantly based on modules and integrations, refer to the [Weaviate documentation](https://weaviate.io/developers/weaviate/current/) for specific configuration details.

### General Notes

- Ensure that both `entrypoint.sh` scripts are executable (`chmod +x entrypoint.sh`) before building your Docker images.
- Adapt the MinIO and Weaviate commands in the `entrypoint.sh` scripts according to your specific needs, such as configuring network settings, security parameters, or enabling specific modules.
- For both services, you might need to configure network settings in `docker-compose.yml` if you're orchestrating multiple containers to ensure they can communicate with each other and with any dependent services.

Remember to consult the official MinIO and Weaviate documentation for the most up-to-date and detailed setup instructions, as configurations can change with new software versions.

## Accessing Specific Versions

To clone a specific version of this project, use the following command, replacing `tag_name` with the desired version tag:

```bash
git clone --branch tag_name --depth 1 https://github.com/cdaprod/minio-weaviate-langchain.git
```

## Setting up an AIOps Self Hosted P2P Development Environment
Here's a step-by-step guide to set up a GitHub repository for deploying AIOps with MinIO and Weaviate using Tailscale, along with Docker and GitHub Actions:

### Step 1: Create a New GitHub Repository
Start by creating a new repository using your existing template. Navigate to [your template repository](https://github.com/Cdaprod/minio-aiops-tailscale/) and use the GitHub interface to generate a new repository from this template.

### Step 2: Create Action Secrets
Set up the necessary GitHub Actions secrets to securely store and manage sensitive data:

- **TS_AUTH_KEY**: Tailscale authentication key to securely connect your devices.
- **TS_SERVE_CONFIG**: Configuration details for Tailscale's serve functionality.
- **TS_CERT_DOMAIN**: Domain for which you are generating HTTPS certificates through Tailscale.

- **MINIO_ROOT_USER**: Username for MinIO admin access.
- **MINIO_ROOT_PASSWORD**: Password for MinIO admin access.

- **DOCKERHUB_USERNAME**: Your Docker Hub username to pull/push images.
- **DOCKERHUB_TOKEN**: Access token for Docker Hub to authenticate API requests.

- **GH_TOKEN**: Personal access token for GitHub to authenticate in GitHub Actions.

These secrets can be added to your repository by going to the repository settings, selecting "Secrets" under "Security," then "Actions," and clicking "New repository secret" for each of the keys and their corresponding values.

### Step 3: Create Self-Hosted Runner
Deploy a self-hosted runner that will handle the CI/CD tasks:

1. **Set Up the Environment**: Prepare a server or a virtual machine where the runner will be installed. This system should have Docker installed if you're planning to build Docker images.
  
2. **Download and Configure the Runner**:
    - Navigate to your repository's settings.
    - Go to "Actions" then "Runners".
    - Click "New runner" and follow the instructions to download, configure, and start the runner using the `GH_TOKEN`.

3. **Script for Runner Setup**:
   ```bash
   mkdir actions-runner && cd actions-runner
   curl -o actions-runner-linux-x64-2.283.3.tar.gz -L https://github.com/actions/runner/releases/download/v2.283.3/actions-runner-linux-x64-2.283.3.tar.gz
   tar xzf actions-runner-linux-x64-2.283.3.tar.gz
   ./config.sh --url https://github.com/<Your-Github-Username>/<Your-Repo-Name> --token <GH_TOKEN>
   ./run.sh
   ```

4. **Start the Runner**:
   Execute the `run.sh` script to start processing jobs.

### Step 4: Configure CI/CD Workflows
Create workflows in your repository to automate the deployment process:
- Define a `.github/workflows/main.yml` file to handle build and deployment tasks triggered by push events or manual triggers.
- Utilize the secrets you set up for authenticating and configuring services like Docker, MinIO, and Tailscale within these workflows.

This setup will ensure that your application is deployed automatically whenever changes are pushed to your repository, leveraging the power of GitHub Actions and your self-hosted runner.

## POC for Deployment of MinIO+Tailscale Network

For the Proof of Concept (POC) #1, titled "Graveyard," which focuses on deploying AIOps with MinIO and Weaviate over Tailscale using Docker and GitHub Actions, you've outlined a clear set of steps and configurations. Below are the refined details and additional explanations to ensure clarity and completeness of the setup process.

### Detailed Steps for POC #1: Graveyard

#### 1. Docker Compose File
This Docker Compose file orchestrates the setup for Tailscale and MinIO containers. The Tailscale container acts as a gateway for MinIO, ensuring all communications are secure within your Tailnet.

```yaml
version: "3.7"
services:
  tailscale-minio:
    image: tailscale/tailscale:latest
    hostname: tailscale-minio
    environment:
      - TS_AUTHKEY=${TS_AUTH_KEY}  # Tailscale authentication key
      - TS_EXTRA_ARGS=--advertise-tags=tag:container
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_USERSPACE=false
    volumes:
      - ${PWD}/tailscale-minio/state:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    restart: unless-stopped

  minio:
    image: minio/minio
    depends_on:
      - tailscale-minio
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    command: server /data
    volumes:
      - minio_data:/data
    network_mode: service:tailscale-minio

volumes:
  minio_data:
```

#### 2. GitHub Actions Workflow
Automate the deployment process via GitHub Actions, providing a CI/CD pipeline that builds, deploys, and verifies the Docker containers.

```yaml
name: Deploy AIOps with Tailscale

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker Environment
        run: docker info

      - name: Build and Push Docker images
        run: |
          docker-compose build
          docker-compose push

      - name: Deploy with Docker Compose
        run: |
          docker-compose up -d

      - name: Verify Deployment
        run: |
          docker-compose ps
```
Place this YAML file under `.github/workflows/deploy.yml` in your GitHub repository.

#### 3. Setting Up Secrets in GitHub Actions
Securely store your Tailscale and MinIO credentials as secrets in GitHub Actions:

- `TS_AUTH_KEY`: Tailscale authentication key.
- `MINIO_ROOT_USER`: MinIO admin username.
- `MINIO_ROOT_PASSWORD`: MinIO admin password.

Navigate to your GitHub repository → Settings → Secrets → Actions, and add these secrets accordingly.

#### 4. Executing Commands
To launch the environment, use:

```bash
docker-compose up -d
```

To check the status of your deployment:

```bash
docker-compose ps
```

This setup will deploy MinIO as your object storage and Tailscale as the secure network overlay, enabling secure, scalable AIOps capabilities. Ensure to test the configuration thoroughly to validate that all components are correctly interacting and accessible within your Tailnet. This foundation will be crucial for scaling to more complex deployments, such as integrating Weaviate for AI-powered data management, which you might cover in future POCs.

## Contributing
We welcome contributions! Please read our contributing guidelines on how to propose changes.
