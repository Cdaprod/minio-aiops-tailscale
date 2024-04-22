# How to Deploy Scalable AIOps Infrastructure with MinIO ver Tailscale’s Secure Network

## Draft 1 - Input

Proof of Concept MinIO
To successfully deploy AIOps with MinIO and Weaviate over Tailscale using Docker and GitHub Actions, you'll need a series of code blocks that can be executed to configure and automate the setup. Here’s a detailed walkthrough:

### 1. Docker Compose File
This Docker Compose file sets up Tailscale and MinIO:

```yaml
version: "3.7"
services:
  tailscale-minio:
    image: tailscale/tailscale:latest
    hostname: tailscale-minio
    environment:
      - TS_AUTHKEY=${TS_AUTH_KEY}  # Tailscale auth key
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
Place this file as `docker-compose.yml` in your project directory.

### 2. GitHub Actions Workflow
This workflow will automate the deployment process using GitHub Actions. It connects to Tailscale, allowing subsequent steps to access nodes in your Tailnet securely.

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
This workflow should be added to your repository in the `.github/workflows` directory as `deploy.yml`.

### 3. Setting Up Secrets in GitHub Actions
You need to configure the following secrets in GitHub Actions to securely store sensitive information:

- `TS_AUTH_KEY`: Your Tailscale authentication key.
- `MINIO_ROOT_USER`: The username for MinIO admin access.
- `MINIO_ROOT_PASSWORD`: The password for MinIO admin access.

Go to your GitHub repository -> Settings -> Secrets -> Actions and add these secrets.

### 4. Executing Commands
To bring up the environment, you would typically execute:

```bash
docker-compose up -d
```

To ensure everything is working as expected, use:

```bash
docker-compose ps
```

This command lists all running containers and shows their status, helping verify that MinIO and Tailscale are up and running.

This comprehensive setup will enable you to deploy AIOps capabilities securely and scalably using MinIO and Weaviate over Tailscale’s network, ensuring data is accessible and protected across your distributed systems.


---

Watching tailscale video, I’m going to provide you with interesting points on the topic for the above POC’s context

- Tailnet is what we call a tailscale network

- Containerizing with tailnet can replace reverse proxies

- access the rest of your tailnet from a deployed tailscale container for hybrid cloud environmentsâ€¦ this is important! Using tailscale encrypted wire guard based tunnels you can tunnel into your local environment to utilize your already existing hardware instead of paying those pesky cloud fees. 

# Key points
- ground breaking for GitOps actions and workflows like CICD 

- no port forwarding, dynamic dns, or fixed firewall rules

- provides their official docker image (exposes several parameters through variables, prob should discuss them and give link)

-  two primary methods for adding containers to your tailnet (three if you count manual deploying and reauthenticating) but two programmatic methods [auth key, and oauth client]

- auth key grants full api access to any client authenticated to use it - auth keys expire 90 day (seperate from node keys) 
- oauth client limits api access via scoping - oauth doesn’t expire meaning that you can handle the key rotations. Claims ownership of specific tag

- tagging - nodes must be owned by user or tag, auth key will assign tag of key creator tag 

# deploying with docker and auth key
environment: 
- TS_AUTHKEY
- TS_STATE_DIR=/var/lib/tailscale

volumes:
- ${PWD}/{container_name}/state:/var/lib/tailscale (persistent directory)
- /dev/net/tun:/dev/net/tun (networking necessity)

cap_add:
- net_admin
- sys_module

minio:
network_mode: service:{service name} ("this is the magic that connects everything together")

# deploying with docker with oauth (additional parameter variables)

environment:
- TS_AUTHKEY={oauth generated secret key) 
- TS_EXTRA_ARGS=--advertise-tags=tags:container (authenticates node as if owned by "container" tag)

In order to make a node not ephemeral the persistent directory needs to be removed from the deployment configuration as well as add "?ephemeral=false" to end of TS_AUTHKEY with no spaces between the key and the "?ephemeral=false".

On Tailscale via magic dns container hostnames are dns resolvable. 

# Linux kernel namespacing

Each container has its own networking context when we add network to specific container we can group containers under that namespace. 

Using namespace tool "nsenter" we can hop into networking namespace context. Docker network creates a new network per stack. With tailscale and network_mode we can utilize it as an upstream overlay network for our containers!

# Putting an app on your tailnet with Serve and Funnel (new parameter variables for the deployment configuration)

Self hosted apps added to tailnet

environment:
- TS_SERVE_CONFIG=/config/{}.json
-  

Tailscale serve lets us provide a configuration file that will tell web requests where to go (like a reverse proxy would do) 

Sample serve json file has syntax errors and such written hastily
```json
{
"TCP" : {
"443":
{
"HTTPS" :
true
}}
"Web": {
"minio.velociraptor-noodlefish.ts.net:443": {
"Handlers"ï¼šï½›
"/"ï¼šï½›
"Proxy": "http://127.0.0.1:9000"
}}}},
"AllowFunnel": {
"minio.velociraptor-noodlefish.ts.net:443": false
}}
``` 

- Funnel would allow me to expose this to the internet with no other changes than changing the AllowFunnel from false to true, suggested to use with ACL. 

- The serve config file will ensure security with LetsEncrypt.

- using {{TS_CERT_DOMAIN}} variable will let us hard coding configuration variables like our "Web": { "${TS_CERT_DOMAIN}â€¦and â€¦"AllowFunnel": ${TS_CERT_DOMAIN}:True 

## While exec in the tailscale container
`tailscale serve status`

`tailscale serve status -json`
- Will print out the ts_config environment variable



Tailscale is absolutely changing the name of the networking game with overlay networks and their hybrid and cross cloud capabilities now with their docker image it finally feels like the final piece to the self-hosted puzzle.

And there we have it we have a self hosted minio app running natively on tailnet with valid https letsencrypt certificate from any local hardware and version controlled using GitOps with GitHub actions that can be configured both for internal and external using tailscale "Funnel". 

With this we can customize our environments with simplicity like never before, whether you have three services deployed in multiple locations across the world, we can encapsulate them with our tailscale p2p overlay network and with this we can grow our devops capabilities to support cross cloud and on premises environments all within the same framework. 

