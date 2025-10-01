---
date_created: 2025-09-28
date_modified: 2025-09-28
document_type: resource
tags:
  - resource
  - domain/software-dev
  - domain/ai
---
[[Resources Dashboard|Resources]] / **[[Docker|Docker]]**
# Docker
**Overview**
Description:: Docker and Docker Compose Commands with Examples

# Docker and Docker Compose Commands with Examples

## Table of Contents

1. [[#Installation and Setup]]
2. [[#Managing Containers]]
3. [[#Useful Docker Commands]]
4. [[#Docker Volume Commands]]
5. [[#Docker Compose Commands]]
6. [[#Persistent Containers with Restart Policy]]
7. [[#Using Environment Variables in Dockerfiles]]
8. [[#Advanced Docker Buildx Usage]]
9. [[#Reinstall and Manage Specific Containers]]

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

The error:

> ❌ `Failed to start docker.service: Unit docker.service not found.`

means that **Docker is not installed** or **not installed correctly** on your Ubuntu server.

---

## ✅ Step-by-step fix on Ubuntu (20.04, 22.04, etc.)

### 1. 🔄 **Uninstall any broken/partial Docker installation**

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

---

### 2. ✅ **Install Docker using the official method**

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
```

Then add Docker’s official GPG key:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Then set up the repository:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

### 3. 📦 **Install Docker Engine**

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
```

---

### 4. 🔥 **Start and enable Docker service**

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify it's running:

```bash
sudo systemctl status docker
```

```bash
docker version
```

🛠️ Рішення: Вимкнути docker.socket
Це найбільш імовірна причина, чому docker.sock не зберігається на диску.

1. Зупини і відключи docker.socket

```bash
sudo systemctl disable --now docker.socket
```

2. Зупини і перезапусти docker

```bash
sudo systemctl stop docker
sudo systemctl start docker
```

3. Перевір сокет

```bash
ls -l /var/run/docker.sock
```

Тепер він повинен з'явитися, і docker ps/docker images працюватимуть.

💡 Що таке docker.socket і чому він заважає?
Це systemd-механізм, який слухає сокет, і запускає dockerd на вимогу, коли перший запит приходить.

Але коли docker.service вже активний — він конфліктує, особливо на нестандартних дистрибутивах або після ручної установки.

---

### 5. 👤 (Optional) Add your user to `docker` group

```bash
sudo usermod -aG docker $USER
```

Then log out and log back in (or `newgrp docker`) to apply the group change.

---

## 🧪 Test it works:

```bash
docker run hello-world
```

---

### Check Docker Daemon Ownership

```bash
ls -l /var/run/docker.sock
```

1. Create the docker Group

   ```bash
   sudo groupadd docker
   ```

2. Add Your User to the docker Group

    ```bash
    sudo usermod -aG docker $USER
    ```

3. Change Ownership of Docker Socket
Ensure that the Docker socket is owned by the docker group:

    ```bash
    sudo chown root:docker /var/run/docker.sock
    ```

4. Set Correct Permissions
Ensure the socket has the correct permissions:

    ```bash
    sudo chmod 660 /var/run/docker.sock
    ```

5. Restart Docker Service

   ```bash
   sudo systemctl restart docker
   ```

6. Verify Group Membership and Ownership
To confirm the changes:

Check if your user is in the docker group:

```bash
groups $USER
```

Confirm the ownership of the Docker socket:

```bash
ls -l /var/run/docker.sock
```

You should see something like this:

```bash
srw-rw---- 1 root docker 0 Jan 30 09:00 /var/run/docker.sock
```

7. Log Out and Back In
For the group changes to take effect, log out and back in or run:

```bash
newgrp docker
```

Now you should be able to run Docker commands without sudo.

## Installation and Setup

### Install Docker and Docker Compose

```bash
sudo apt install docker docker-compose
```

### Add User to Docker Group

```bash
sudo usermod -aG docker pi
```

### Start Docker Daemon

```bash
systemctl start docker
```

### Enable Docker and Containerd to Start on Boot

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### Disable Docker and Containerd from Starting on Boot

```bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Managing Containers

### Download Portainer

```bash
sudo docker pull portainer/portainer-ce:linux-arm
```

### Start Portainer Container

```bash
sudo docker run --restart always -d -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:linux-arm
```

_Portainer credentials: User - `admin`, Password - `MishaDimaVika`_

### Start `web_recognition` Container with Volume Mapping

```bash
docker run --restart always -d --name=web_face \
  --mount source=data,destination=/root/face_recognition/examples/data \
  -p 5001:5001 web_recognition:210515
```

### Run a `web_recognition` Container Interactively

```bash
docker run -it --name=web_face \
  --mount source=data,destination=/root/face_recognition/examples/data \
  -p 5001:5001 andriiradchenko/web_recognition:210515
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Useful Docker Commands

### List All Images

```bash
docker images
```

### Run a Container in Daemon Mode with Port Forwarding

```bash
docker run -d -p 1234:8080 tomcat
```

### List Running Containers

```bash
docker ps
```

### Stop and Remove a Container

```bash
docker rm [container_id]
```

### Remove an Image

```bash
docker rmi tomcat
```

### Build an Image from a Dockerfile

```bash
docker build -t web_recognition_v01:amd64 . --load
```

### Remove Intermediate Images in a Multistage Build

```bash
docker image prune -f --filter label=stage=build-step
```

### Stop and Remove All Containers

```bash
docker rm -f $(docker ps -aq)
```

### Run a Shell Inside a Container

```bash
docker exec -it [container_id] /bin/bash
```

### Exit a Container Shell

```bash
exit
```

### Commit a Running Container as an Image

```bash
docker commit [container_id] newimage_v2:latest
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Docker Volume Commands

### List Volumes

```bash
docker volume ls
```

### Inspect a Volume

```bash
docker volume inspect data
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Docker Compose Commands

### Build project with different .env

```bash
docker-compose --env-file .env up -d
docker-compose --env-file .env.vpn up -d --force-recreate
```

### Build Containers Without Using the Cache

```bash
docker-compose build --no-cache
```

### See container log

```bash
docker-compose logs nestjs
docker logs nestjs
```

### Build and Start Containers

```bash
docker-compose up
```

### Start in Detached Mode (No Logs)

```bash
docker-compose up -d
```

### Stop and Remove Containers

```bash
docker-compose down
```

### Stop and Remove Containers with Associated Images

```bash
docker-compose down --rmi all
```

### Start/Stop Specific Services

```bash
docker-compose start

docker-compose stop
```

### View Running Services

```bash
docker-compose ps
```

### View Docker Compose Images

```bash
docker-compose images
```
No, **you do not need to run `docker-compose down`** before rebuilding `nestjs` and `nextjs`.

In fact, **you *should not*** call `down` if you want to keep your `Postgres` and `MinIO` data intact and avoid stopping them.

Here’s exactly what to do:

---

**I have docker-compose file for build 4 containers: nestjs nextjs Postgres and minio. I’ve changed code for my nestjs and nextjs projects. How to rebuild all project but keep minio and Postgres untouched?**

### ✅ If your project is already running with `docker-compose up -d`:

Just run:

```bash
docker-compose build nestjs nextjs
docker-compose up -d --no-deps nestjs nextjs
```

This will:

1. Rebuild the `nestjs` and `nextjs` containers from updated source code.
2. Restart only those two containers.
3. Leave `Postgres` and `MinIO` **running and untouched**.

---

### 🚫 Why not `docker-compose down`?

Because `docker-compose down`:

* **Stops all containers**.
* **Removes** containers, **networks**, and **non-volume data**.
* Can disrupt services like Postgres and MinIO if they're not using volumes or persistent data configuration.

Yes, you **can absolutely use** the `--env-file` flag together with:

```bash
docker-compose build nestjs nextjs
docker-compose up -d --no-deps nestjs nextjs
```


---

### ✅ But if you're using CLI-based `.env` injection:

You can also do this:

```bash
docker-compose --env-file ./my-env-file.env build nestjs nextjs
docker-compose --env-file ./my-env-file.env up -d --no-deps nestjs nextjs
```

> This overrides or supplements what’s in `env_file:` blocks.

---

### ⚠️ Important notes:

1. `--env-file` applies only to the Compose CLI, **not to the containers** directly.
2. If your Dockerfile or app expects env vars at build time, make sure you **pass them in `build.args:`** or use `.env` files properly.
3. You can only use one `--env-file` per command. If you need multiple files, merge them first.

---

### 🧪 Example command

```bash
docker-compose --env-file .env.dev build nestjs nextjs
docker-compose --env-file .env.dev up -d --no-deps nestjs nextjs
```

This ensures that both build and run phases use the same environment.

Let me know if you want help configuring `env_file:` vs `environment:` vs `build.args` for your stack — they behave slightly differently but can work together.


---

Let me know if you also want to **watch files and auto-reload** in `nestjs`/`nextjs` using `nodemon` or `next dev`, or handle live rebuilds without even rebuilding containers manually.


[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Persistent Containers with Restart Policy

To ensure containers restart on host reboot, use the `restart: unless-stopped`
policy in `docker-compose.yml`:

```yaml
services:
  nextjs:
    build:
      context: ./quiz-admin
      args:
        NEXT_PUBLIC_S3_END_POINT: minio-compose
        NEXT_PUBLIC_S3_PORT: 9000
    container_name: nextjs
    restart: unless-stopped
    ports:
      - '3000:3000'
    environment:
      NEXT_PUBLIC_S3_END_POINT: minio-compose
      NEXT_PUBLIC_S3_PORT: 9000
      NEXT_PUBLIC_S3_ACCESS_KEY: minio
      NEXT_PUBLIC_S3_SECRET_KEY: miniominio
      NEXT_PUBLIC_S3_BUCKET_QUESTIONS: questions
      NEXT_PUBLIC_S3_BUCKET_PLAYERS: players
    working_dir: /app
    depends_on:
      - minio-compose
    networks:
      - quiz-network
```

Start the stack:

```bash
docker-compose up -d
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Using Environment Variables in Dockerfiles

### Add Environment Variables in the `Dockerfile`

```dockerfile
# Build arguments
ARG NEXT_PUBLIC_S3_END_POINT
ARG NEXT_PUBLIC_S3_PORT

# Export environment variables
ENV NEXT_PUBLIC_S3_END_POINT=$NEXT_PUBLIC_S3_END_POINT
ENV NEXT_PUBLIC_S3_PORT=$NEXT_PUBLIC_S3_PORT
```

### Pass Variables During Build

```bash
docker-compose build --build-arg NEXT_PUBLIC_S3_END_POINT=minio-compose \
  --build-arg NEXT_PUBLIC_S3_PORT=9000
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Advanced Docker Buildx Usage

### Create a Multi-Platform Builder

```bash
docker buildx create --platform linux/arm64,linux/arm/v7
```

### Use a Custom Builder

```bash
docker buildx use [OPTIONS] NAME
```

### Run a QEMU-Enabled Multiarch Builder

```bash
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

### Build Multi-Platform Images

```bash
docker buildx build --platform linux/arm64 -t user/repo --no-cache --pull .
```

### Example: Build for Specific Platforms

```bash
# Build for ARMv7 (32-bit)
docker buildx build --platform linux/arm/v7 -t web_recognition:armv7 . --load

# Build for ARM64 (64-bit)
docker buildx build --platform linux/arm64 -t web_recognition:arm64 .
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]

---

## Reinstall and Manage Specific Containers

### Reinstall `web_face` Container

#### Pull Updated Image

```bash
docker pull andriiradchenko/web_recognition:armv702
```

#### Run with Volume Mapping

```bash
docker run --restart always -d --name=web_face \
  --mount source=data,destination=/root/face_recognition/examples/data \
  -p 5001:5001 web_recognition:armv702
```

#### Persistent Data Example

```bash
docker run --restart always -d --name=web_face -v face_data:/root/face_recognition/examples \
  -p 5001:5001 andriiradchenko/web_recognition
```

[[#Docker and Docker Compose Commands with Examples|Back to Top]]




---
[[Resources Dashboard|Resources]] / **[[Docker|Docker]]**