![Progress](https://img.shields.io/badge/Progress-16%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 16 — Docker Zero to Hero

## 📝 Topic: Containers, Docker Architecture, Lifecycle & First Image Build
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 23, 2026

---

## 🎯 Learning Objectives
* Define what a container is in plain terms and in technical terms.
* Understand why container base images are ~100x smaller than VM images.
* Know what files a container carries vs what it borrows from the host OS.
* Understand Docker architecture — daemon, client, registry, and their relationships.
* Execute the three-step Docker lifecycle: `build` → `run` → `push`.
* Install Docker on Ubuntu EC2, start the daemon, and fix the permission denied error.
* Build, verify, run, and push a first Docker image to DockerHub.

---

## 📦 Part 1 — What is a Container?

### The Technical Definition

> *"A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another."*

### The Simple Definition

```
Container = Application
          + Application libraries required to run it
          + Minimum system dependencies

Nothing more. Nothing less.
```

---

## ⚖️ Part 2 — Containers vs Virtual Machines

| Aspect | Virtual Machines | Containers |
|---|---|---|
| **Resource Utilization** | Full OS + hypervisor — resource-heavy | Shared host kernel — lightweight and fast |
| **Portability** | Needs a compatible hypervisor to run | Runs on any host with a compatible OS |
| **Security** | Higher isolation — each VM has its own OS | Lower isolation — shared host kernel |
| **Management** | Heavier — large images, slow boot | Easier — small images, instant start |

---

## 🪶 Part 3 — Why Are Containers Lightweight?

Containers share the host OS kernel. They do not bundle a full operating system inside themselves.

### The Numbers

```
Official Ubuntu container base image:  ~22 MB
Official Ubuntu VM image:              ~2.3 GB

Container is ~100x smaller than the equivalent VM image.
```

### What a Container Base Image Contains

```
/bin   → binary executables (ls, cp, ps)
/sbin  → system binaries (init, shutdown)
/etc   → configuration files for system services
/lib   → library files used by the binaries
/usr   → user-related utilities, apps, documentation
/var   → variable data — logs, spool files, temp files
/root  → home directory of the root user
```

### What Containers Borrow from the Host OS

```
Host kernel        → handles system calls (CPU, memory, I/O access)
Networking stack   → provides network connectivity
Namespaces         → isolates filesystem, process IDs, network per container
cgroups            → limits CPU, memory, I/O usage per container
```

> **Key point:** A container uses host resources through namespaces and cgroups — but changes inside the container do not affect the host or other containers. Isolation is maintained through the kernel, not through a separate OS.

---

## 🐳 Part 4 — What is Docker?

> *"Containerization is a concept or technology. Docker implements containerization."*

Docker is a **containerization platform** that provides tools to:
* Build container images from a `Dockerfile`
* Run images as containers
* Push images to registries (DockerHub, Quay.io, ECR, etc.)

---

## 🏗️ Part 5 — Docker Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Docker Client (CLI)                │
│          docker build / docker run / docker push    │
└───────────────────────┬─────────────────────────────┘
                        │ Docker API calls
                        ▼
┌─────────────────────────────────────────────────────┐
│              Docker Daemon (dockerd)                │
│   The brain of Docker. Manages images, containers,  │
│   networks, and volumes. Listens for API requests.  │
│                                                     │
│   ⚠️ If dockerd crashes → ALL containers affected  │
└──────────────┬───────────────────────┬──────────────┘
               │                       │
               ▼                       ▼
     ┌──────────────────┐    ┌──────────────────────┐
     │    Containers     │    │  Docker Registry     │
     │  (running images) │    │  DockerHub / ECR     │
     └──────────────────┘    │  (stores images)     │
                              └──────────────────────┘
```

### Key Components

| Component | Role |
|---|---|
| **Docker Daemon (`dockerd`)** | The core engine — manages all Docker objects (images, containers, networks, volumes). If it dies, Docker is non-functional. |
| **Docker Client (`docker`)** | The CLI tool users interact with. Sends commands to `dockerd` via the Docker API. |
| **Docker Desktop** | GUI application for Mac/Windows/Linux. Bundles the daemon, client, Docker Compose, and Kubernetes. |
| **Docker Registry** | Stores and distributes images. DockerHub is the default public registry. Can self-host a private registry. |
| **Dockerfile** | A script with step-by-step instructions to build a Docker image. |
| **Image** | A read-only, layered template. Each `Dockerfile` instruction creates one layer. Only changed layers are rebuilt. |
| **Container** | A running instance of an image. Ephemeral — data inside is lost when the container is removed. |

---

## 🔄 Part 6 — Docker Lifecycle

```
Dockerfile  ──[docker build]──▶  Image  ──[docker run]──▶  Container
                                    │
                              [docker push]
                                    ▼
                              Docker Registry
                            (DockerHub / ECR)
```

**The three commands that drive everything:**

```bash
docker build   # Dockerfile → Image
docker run     # Image → Running Container
docker push    # Image → Registry (shareable)
```

---

## 🛠️ Part 7 — Installing Docker on Ubuntu EC2

### Install

```bash
sudo apt update
sudo apt install docker.io -y
```

### Verify Installation

```bash
docker run hello-world
```

### ❌ Common Error: Permission Denied

```
docker: Got permission denied while trying to connect to the Docker daemon socket
at unix:///var/run/docker.sock
```

**Two possible causes:**

```
1. Docker daemon is not running
2. Your user does not have access to run docker commands
```

### Fix 1 — Start the Docker Daemon

```bash
# Check daemon status
sudo systemctl status docker

# Start the daemon if it's not running
sudo systemctl start docker

# Enable it to start on boot
sudo systemctl enable docker
```

### Fix 2 — Grant User Access to Docker

```bash
# Add your user to the docker group
sudo usermod -aG docker ubuntu
# Replace 'ubuntu' with your actual username

# ⚠️ IMPORTANT: Log out and log back in for the change to take effect
# The group membership only applies to new sessions
```

### Verify Everything Works

```bash
docker run hello-world
```

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## 🏗️ Part 8 — Building Your First Docker Image

### Step 1: Clone the Repository

```bash
git clone https://github.com/iam-veeramalla/Docker-Zero-to-Hero
cd Docker-Zero-to-Hero/examples
```

### Step 2: Login to DockerHub

```bash
docker login
```

```
Username: yourusername
Password:
Login Succeeded
```

> ⚠️ Docker warns that the password is stored unencrypted in `~/.docker/config.json`. Use a **credential helper** in production to avoid this.

### Step 3: Build the Image

```bash
docker build -t yourusername/my-first-docker-image:latest .
```

**What happens during the build:**

```
Step 1/6 : FROM ubuntu:latest        → pull base image
Step 2/6 : WORKDIR /app              → set working directory
Step 3/6 : COPY . /app               → copy source files
Step 4/6 : RUN apt-get update && apt-get install -y python3 python3-pip
Step 5/6 : ENV NAME World            → set environment variable
Step 6/6 : CMD ["python3", "app.py"] → define startup command

Successfully built 960d37536dcd
Successfully tagged yourusername/my-first-docker-image:latest
```

Each `Step` = one Dockerfile instruction = one image layer. Changed layers are rebuilt; unchanged layers are served from cache.

### Step 4: Verify the Image

```bash
docker images
```

```
REPOSITORY                        TAG       IMAGE ID       CREATED          SIZE
yourusername/my-first-docker-image latest   960d37536dcd   26 seconds ago   467MB
ubuntu                            latest    58db3edaf2be   13 days ago      77.8MB
hello-world                       latest    feb5d9fea6a5   16 months ago    13.3kB
```

### Step 5: Run the Container

```bash
docker run -it yourusername/my-first-docker-image
```

```
Hello World
```

### Step 6: Push to DockerHub

```bash
docker push yourusername/my-first-docker-image
```

```
896818320e80: Pushed
b8088c305a52: Pushed
69dd4ccec1a0: Pushed
c5ff2d88f679: Mounted from library/ubuntu
latest: digest: sha256:6e498... size: 1157
```

Each layer is pushed independently. If a layer already exists in the registry (`Mounted from library/ubuntu`), it is reused — not re-uploaded.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Container** | App + required libraries + minimum system dependencies — shares host kernel |
| **VM** | Full virtual machine with its own OS — runs on a hypervisor |
| **Namespace** | Linux kernel feature that provides per-container isolation (filesystem, PID, network) |
| **cgroups** | Linux kernel feature that limits and controls resource usage per container |
| **Docker Daemon (`dockerd`)** | The core Docker engine — manages all objects, single point of failure |
| **Docker Client** | The `docker` CLI — sends API commands to `dockerd` |
| **Docker Registry** | Stores and distributes container images (DockerHub, ECR, Quay.io) |
| **Dockerfile** | Step-by-step script that builds a Docker image |
| **Image** | A read-only, layered snapshot — the blueprint for containers |
| **Container** | A running instance of an image — ephemeral by default |
| **Layer** | A single Dockerfile instruction baked into the image — cached independently |
| **`docker build`** | Creates a Docker image from a Dockerfile |
| **`docker run`** | Creates and starts a container from an image |
| **`docker push`** | Uploads an image to a registry |
| **`docker images`** | Lists all locally available images |
| **`docker login`** | Authenticates with a Docker registry |
| **`usermod -aG docker`** | Adds a user to the docker group — grants permission to run docker commands |
| **`systemctl start docker`** | Starts the Docker daemon service |
| **`-it` flag** | Interactive + TTY — allows terminal input/output with the container |
| **DockerHub** | Docker's default public image registry |

---

## 📂 Summary of Tasks
- ✅ Defined: Container — app + libraries + minimum dependencies + shared kernel.
- ✅ Understood: Why container base images are ~100x smaller than VM images.
- ✅ Mapped: What containers carry (app, libs) vs what they borrow (kernel, namespaces, cgroups).
- ✅ Understood: Docker architecture — daemon, client, registry, image, container.
- ✅ Understood: The three-command lifecycle — `build` → `run` → `push`.
- ✅ Installed: Docker on Ubuntu EC2 and started the daemon.
- ✅ Fixed: The `permission denied` error — daemon check + `usermod -aG docker`.
- ✅ Built: First Docker image from a Dockerfile.
- ✅ Ran: First container and confirmed output.
- ✅ Pushed: Image to DockerHub.

---

## 💡 My Takeaway

The permission denied error after installing Docker is the kind of thing that stops beginners for an hour. The fix is two commands — check the daemon with `systemctl status docker`, add the user to the docker group with `usermod -aG docker ubuntu` — but you have to know to look for it. The log-out-and-back-in requirement after `usermod` is another silent trap. These are the friction points documentation often skips.

The layer caching model was the other thing that clicked. Each Dockerfile instruction is a layer. Each layer is cached independently. When you rebuild, Docker only re-executes instructions below the first changed layer. This is why Dockerfile instruction order matters: expensive steps (like `RUN apt-get install`) should come before frequently changing steps (like `COPY . .`). Get the order wrong and you reinstall dependencies on every single build.

---

## 📈 Next Up
**Day 16:** Docker Multi-Stage Builds & Distroless Images — reducing a 861 MB image to 1.83 MB, and why production images must never carry build tools.

---

## 🔗 Resources
* [Docker-Zero-to-Hero Repository](https://github.com/iam-veeramalla/Docker-Zero-to-Hero)
* [Docker Installation Docs](https://docs.docker.com/get-docker/)
* [DockerHub](https://hub.docker.com/)
* [Docker Architecture Docs](https://docs.docker.com/get-started/overview/#docker-architecture)
* [Linux Namespaces](https://www.man7.org/linux/man-pages/man7/namespaces.7.html)
* [Linux cgroups](https://www.man7.org/linux/man-pages/man7/cgroups.7.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*