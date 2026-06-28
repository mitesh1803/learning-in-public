![Progress](https://img.shields.io/badge/Progress-17%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 17 — Docker Advanced: Multi-Stage Builds, Distroless Images & Volumes

## 📝 Topic: Image Optimization + Persistent Storage for Containers
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 25, 2026

---

## 🎯 Learning Objectives
* Understand why standard Docker builds produce bloated, insecure images.
* Implement Multi-Stage Builds to separate build-time and runtime dependencies.
* Understand Distroless images — what they exclude and why that matters for security.
* Reduce a Golang image from 861 MB to 1.83 MB using `scratch`.
* Understand why containers are ephemeral and why data persistence requires extra work.
* Distinguish Bind Mounts from Docker Volumes — and know when to use each.
* Use all Docker volume CLI commands: `create`, `ls`, `inspect`, `rm`.
* Use `--mount` syntax for readable, verbose volume configuration.

---

## ⚠️ Part 1 — The Problem with Standard Docker Builds

### What a Naive Dockerfile Includes

```dockerfile
# Naive approach
FROM ubuntu:latest
RUN apt-get update && apt-get install -y golang
WORKDIR /app
COPY . .
RUN go build -o my-app .
CMD ["./my-app"]
```

**What this image contains at runtime:**

```
ubuntu base OS utilities     → ls, cat, bash, sh...
apt package manager          → apt, apt-get, dpkg...
golang compiler + toolchain  → go, gofmt, compiler cache...
build caches                 → intermediate object files
your actual application      → my-app (a few MB)

Total image size: ~861 MB
Your app size:    ~8 MB
Bloat:            853 MB of tools your app never uses at runtime
```

**Two problems this creates:**

| Problem | Detail |
|---|---|
| **Size** | Pulling an 861 MB image on every deployment is slow, expensive, and wastes registry storage |
| **Attack surface** | Every tool in the image is a potential vector — if an attacker gets into the container, `bash`, `curl`, `apt` are all available to them |

> **The core insight:** Build tools are needed to compile. Runtime tools are needed to run. They are not the same set. A standard build image conflates them.

---

## 🏗️ Part 2 — Multi-Stage Docker Builds

### The Concept

```
Stage 1 (Build Stage):
  → Use a heavy image with all build tools
  → Compile the application
  → Produce a binary/artifact

Stage 2 (Final Stage):
  → Use a minimal base image
  → COPY only the compiled artifact from Stage 1
  → Everything from Stage 1 (compiler, tools, caches) is LEFT BEHIND
```

### Multi-Stage Dockerfile — Golang Example

```dockerfile
# ─── Stage 1: Build ───────────────────────────────────────
FROM golang:1.21 AS builder

WORKDIR /app

# Copy and download dependencies first (layer caching)
COPY go.mod go.sum ./
RUN go mod download

# Copy source and build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o my-app .

# ─── Stage 2: Final Runtime Image ─────────────────────────
FROM ubuntu:22.04

WORKDIR /app

# Copy ONLY the compiled binary from the build stage
COPY --from=builder /app/my-app .

# No Go compiler. No build tools. No caches.
CMD ["./my-app"]
```

**What `COPY --from=builder` does:**

```
Stage 1 (builder):           Stage 2 (final):
  golang compiler   ✗          
  go toolchain      ✗          my-app binary ✅  ← only this crosses over
  build cache       ✗          
  source code       ✗          
  go modules cache  ✗          
```

### Size Comparison — Standard vs Multi-Stage

```
Standard build:       861 MB   (compiler + tools + app)
Multi-Stage (ubuntu): ~80 MB   (ubuntu base + app binary only)
Multi-Stage (scratch): 1.83 MB (nothing + app binary only)
```

### Multi-Stage — Python Example

```dockerfile
# ─── Stage 1: Build ───────────────────────────────────────
FROM python:3.11 AS builder

WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# ─── Stage 2: Final ───────────────────────────────────────
FROM python:3.11-slim

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
COPY . .

ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

```
python:3.11 full image:  ~1.0 GB
python:3.11-slim result: ~150 MB
```

---

## 🔒 Part 3 — Distroless Images

### What is a Distroless Image?

> *"Distroless images contain only your application and its runtime dependencies — no shells, no package managers, no OS utilities."*

```
Standard Ubuntu image contains:
  bash, sh, dash         ← shells
  apt, dpkg              ← package managers
  ls, cat, grep, curl    ← system utilities
  ping, netstat          ← network tools
  ...hundreds more

Distroless image contains:
  your application binary
  required runtime libraries (glibc, etc.)
  CA certificates (for HTTPS)
  timezone data
  That's it.
```

### Security Benefit — The Attack Surface

```
If an attacker exploits a vulnerability and gets RCE (Remote Code Execution) in your container:

Standard image:
  They have bash → they can explore the filesystem
  They have curl → they can download malware
  They have apt  → they can install attack tools
  
Distroless image:
  No shell       → can't run interactive commands
  No curl        → can't download anything
  No apt         → can't install anything
  
Attacker gets in → finds nothing useful → impact dramatically limited
```

### Google's Distroless Images

```dockerfile
# Java application with distroless
FROM eclipse-temurin:17-jdk AS builder
WORKDIR /app
COPY . .
RUN ./gradlew build

FROM gcr.io/distroless/java17-debian11
COPY --from=builder /app/build/libs/app.jar /app.jar
CMD ["/app.jar"]
```

### The Ultimate Minimal: `scratch`

`scratch` is Docker's empty base image — literally nothing. Zero bytes. Used for statically compiled languages like Go.

```dockerfile
# ─── Stage 1: Build ───────────────────────────────────────
FROM golang:1.21 AS builder

WORKDIR /app
COPY . .

# CGO_ENABLED=0 → statically linked binary, no external lib dependencies
RUN CGO_ENABLED=0 GOOS=linux go build -o my-app .

# ─── Stage 2: Scratch — absolute minimum ──────────────────
FROM scratch

COPY --from=builder /app/my-app /my-app
CMD ["/my-app"]
```

**The result:**

```
Standard golang build:   861 MB
Multi-stage + scratch:   1.83 MB

Reduction: ~470x smaller
```

Why does this work for Go? Go can compile to a **statically linked binary** — a single executable that carries all its dependencies inside itself. It needs no OS libraries, no runtime environment, no shell. The `scratch` image provides no filesystem at all — just enough to run that one binary.

### Distroless Comparison Table

| Base Image | Size | Shell | Package Manager | Use Case |
|---|---|---|---|---|
| `ubuntu:latest` | ~77 MB | ✅ bash | ✅ apt | Development, debugging |
| `ubuntu:22.04-slim` | ~29 MB | ✅ bash | ✅ apt | Lighter general purpose |
| `gcr.io/distroless/base` | ~20 MB | ❌ | ❌ | Most production apps |
| `gcr.io/distroless/static` | ~2 MB | ❌ | ❌ | Statically compiled apps |
| `scratch` | 0 MB | ❌ | ❌ | Go, Rust — fully static binaries |

---

## 🎤 Part 4 — Interview Answer: Production Image Issues

**Q: "Have you dealt with any Docker image-related production issues?"**

> "Yes — we had images reaching over 1 GB for relatively simple services, which was causing slow pull times in our CI/CD pipeline and creating unnecessary vulnerability exposure. I transitioned the team to multi-stage builds, separating build-time dependencies from runtime dependencies. For our Go services, we moved the final stage to `scratch`, which reduced image sizes by over 400x. For our Python and Java services, we adopted distroless base images. Both changes solved two problems simultaneously: image size dropped dramatically, reducing deployment time and registry costs, and the attack surface shrunk because the final images contain none of the OS utilities an attacker could leverage after a breach."

---

## 💾 Part 5 — The Ephemeral Container Problem

### Why Containers Lose Data

```
Container lifecycle:
  docker run myapp → container starts
                   → application writes logs to /var/log/app/
                   → application writes data to /var/lib/data/
  docker stop myapp → container stops
  docker rm myapp   → container removed

  ALL data inside the container is GONE.
  Logs: gone. Database files: gone. Uploaded files: gone.
```

**Why containers are designed this way:**

Containers are meant to be **stateless and replaceable**. Kubernetes kills and restarts them. CI/CD pipelines spin up fresh containers for every build. The ephemeral model is a feature — but it requires explicitly externalizing any state that must survive.

```
Stateless (container handles): app logic, request processing, computation
Stateful (must persist):        databases, logs, user uploads, certificates
```

---

## 🔗 Part 6 — Bind Mounts

### What is a Bind Mount?

A bind mount maps a **specific path on the host** directly into the container.

```
Host filesystem:          Container filesystem:
/home/ubuntu/configs/ ←→  /app/config/
/home/ubuntu/logs/    ←→  /var/log/app/
```

Whatever exists at the host path is visible inside the container. Changes made inside the container appear on the host immediately, and vice versa.

### Bind Mount Syntax

```bash
# Using -v (short form)
docker run -d \
  -v /host/path:/container/path \
  my-app

# Using --mount (verbose, recommended)
docker run -d \
  --mount type=bind,source=/home/ubuntu/configs,target=/app/config \
  my-app
```

### Real-World Use Cases

```bash
# Share config files with a running container
docker run -d \
  --mount type=bind,source=/etc/nginx/nginx.conf,target=/etc/nginx/nginx.conf \
  nginx

# Mount source code for live development (changes reflected immediately)
docker run -d \
  --mount type=bind,source=$(pwd),target=/app \
  node:20 npm run dev

# Persist logs to host for log aggregation
docker run -d \
  --mount type=bind,source=/var/log/myapp,target=/var/log/app \
  my-app
```

### ⚠️ Bind Mount Limitations

```
Problem 1 — Host dependency:
  The specific path must exist on every host the container runs on
  → /home/ubuntu/configs on laptop ≠ /home/ec2-user/configs on production

Problem 2 — Portability:
  Bind mounts are tightly coupled to the host filesystem structure
  → Hard to back up, migrate, or replicate across environments

Problem 3 — Permissions:
  Container processes may not have the right permissions on the host path
```

---

## 📦 Part 7 — Docker Volumes

### What is a Docker Volume?

A Docker-managed storage area stored at `/var/lib/docker/volumes/` on the host. Docker creates, manages, and abstracts the storage — you never reference a specific host path.

```
Host filesystem:
  /var/lib/docker/volumes/
    myapp-data/
      _data/          ← actual data stored here
        db.sqlite
        uploads/

Container sees:
  /var/lib/data/      ← mounted from myapp-data volume
```

### Volume Lifecycle Commands

```bash
# Create a named volume
docker volume create myapp-data

# List all volumes
docker volume ls
# DRIVER    VOLUME NAME
# local     myapp-data
# local     postgres-data

# Inspect a volume — see actual host path and metadata
docker volume inspect myapp-data
# [
#   {
#     "Name": "myapp-data",
#     "Driver": "local",
#     "Mountpoint": "/var/lib/docker/volumes/myapp-data/_data",
#     "CreatedAt": "2026-06-24T09:15:22Z"
#   }
# ]

# Remove a volume
# ⚠️ Container must be stopped and removed first
docker stop myapp && docker rm myapp
docker volume rm myapp-data
```

### Using Volumes with `--mount` (Recommended)

```bash
# Attach a named volume to a container
docker run -d \
  --name myapp \
  --mount source=myapp-data,target=/var/lib/data \
  my-app

# If the volume doesn't exist yet, Docker creates it automatically
docker run -d \
  --name postgres \
  --mount source=postgres-data,target=/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

### Volume Advantages Over Bind Mounts

| Feature | Bind Mount | Docker Volume |
|---|---|---|
| **Host path dependency** | ✅ Tightly coupled to host path | ❌ Docker manages location |
| **Portability** | Low — breaks when host changes | High — works on any Docker host |
| **Backup** | Manual — find the host path | Simple — `docker volume` commands |
| **Multiple containers** | Complex — coordinate paths | Easy — name the volume, share it |
| **External storage** | Not supported | ✅ NFS, cloud storage (AWS EFS, etc.) |
| **Docker CLI management** | ❌ | ✅ create, ls, inspect, rm |
| **Best for** | Dev config sharing, source code mounts | Databases, persistent app data |

### Sharing a Volume Between Multiple Containers

```bash
# Container 1: writes data
docker run -d \
  --name writer \
  --mount source=shared-data,target=/data \
  my-writer-app

# Container 2: reads the same data
docker run -d \
  --name reader \
  --mount source=shared-data,target=/data \
  my-reader-app

# Both containers see the same /data directory
```

### External Volume Drivers

```bash
# Mount an NFS share as a Docker volume
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/nfs/share \
  nfs-volume

# AWS EFS (Elastic File System) — requires the efs-utils driver
docker volume create \
  --driver amazon-efs \
  efs-volume
```

---

## 📋 Part 8 — `-v` vs `--mount`: Which to Use

```bash
# -v shorthand (common but harder to read)
docker run -d -v myapp-data:/var/lib/data my-app
docker run -d -v /host/path:/container/path my-app   # bind mount

# --mount verbose (recommended — explicitly states type, source, target)
docker run -d --mount type=volume,source=myapp-data,target=/var/lib/data my-app
docker run -d --mount type=bind,source=/host/path,target=/container/path my-app
```

**Why `--mount` is preferred:**

```
-v /host/path:/container  → ambiguous: is this a bind mount or named volume?
                            ambiguous: which direction does data flow?

--mount type=bind,source=/host/path,target=/container
         ↑ clear  ↑ clear              ↑ clear
```

The `--mount` syntax is self-documenting. Anyone reading the command knows immediately whether it's a bind mount or a volume, what the source is, and what the target is. `-v` requires mental parsing of the colon syntax.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Multi-Stage Build** | A Dockerfile with multiple `FROM` stages — build in one stage, copy artifact to a minimal final stage |
| **`COPY --from=`** | Copies files from a named build stage into the current stage |
| **`AS builder`** | Names a build stage for reference in later `COPY --from=` instructions |
| **`CGO_ENABLED=0`** | Disables Go's C bridge — produces a fully statically linked binary |
| **Distroless** | Container images containing only the app and runtime — no shell, no package manager |
| **`scratch`** | Docker's completely empty base image — zero bytes, used for fully static binaries |
| **Attack Surface** | The set of tools and entry points available to an attacker inside a container |
| **Static Binary** | A compiled executable that carries all its dependencies — needs no OS libraries at runtime |
| **Ephemeral** | Short-lived — containers do not persist data between runs by default |
| **Bind Mount** | Maps a specific host filesystem path into a container |
| **Docker Volume** | Docker-managed persistent storage — stored in `/var/lib/docker/volumes/` |
| **Named Volume** | A volume created with `docker volume create` — referenced by name, not path |
| **`docker volume create`** | Creates a named Docker volume |
| **`docker volume ls`** | Lists all Docker volumes on the host |
| **`docker volume inspect`** | Shows metadata and host path for a volume |
| **`docker volume rm`** | Deletes a volume (container must be stopped and removed first) |
| **`-v` flag** | Short-form volume/bind mount syntax for `docker run` |
| **`--mount` flag** | Verbose, explicit volume/bind mount syntax — recommended for readability |
| **Volume Driver** | A plugin enabling volumes to be backed by external storage (NFS, AWS EFS, etc.) |
| **`/var/lib/docker/volumes/`** | Default host path where Docker stores all named volume data |

---

## 📂 Summary of Tasks
- ✅ Understood: Why standard Docker builds bloat image size with build-time tools.
- ✅ Implemented: Multi-Stage Build — separate builder and runtime stages in one Dockerfile.
- ✅ Used: `COPY --from=builder` to bring only the compiled artifact into the final stage.
- ✅ Understood: Distroless images — no shell, no package manager, smaller attack surface.
- ✅ Used: `scratch` as the final stage for a Go app — 861 MB → 1.83 MB.
- ✅ Understood: Why containers are ephemeral and when that becomes a problem.
- ✅ Understood: Bind Mounts — host path mapped directly into container, dev-focused.
- ✅ Understood: Docker Volumes — Docker-managed, portable, production-focused.
- ✅ Practiced: `docker volume create`, `ls`, `inspect`, `rm`.
- ✅ Used: `--mount` syntax for explicit, readable volume configuration.
- ✅ Understood: When to prefer volumes over bind mounts for production workloads.

---

## 💡 My Takeaway

The 861 MB → 1.83 MB reduction for a Go application is a real number, not a theoretical possibility. The entire change is two additions to the Dockerfile: name the first stage `AS builder` and add a `FROM scratch` final stage with a single `COPY --from=builder`. Two lines of code. 470x size reduction. That's the kind of leverage that makes Multi-Stage Builds non-negotiable for production images.

On volumes: the distinction between bind mounts and Docker volumes is really a distinction between dev and production patterns. Bind mounts are great for sharing your local source code into a container during development — you edit on the host, the container picks it up instantly. But bind mounts are tied to specific host paths, which makes them fragile in production where hosts change. Named volumes let Docker manage the "where" — you just say "this container needs storage called `postgres-data`" and Docker handles the rest.

---

## 📈 Next Up
**Day 18:** Docker Networking — bridge networks, host networking, container-to-container communication, and Docker Compose for multi-container orchestration.

---

## 🔗 Resources
* [Docker Multi-Stage Build Docs](https://docs.docker.com/build/building/multi-stage/)
* [Google Distroless Images](https://github.com/GoogleContainerTools/distroless)
* [Docker Volumes Docs](https://docs.docker.com/storage/volumes/)
* [Docker Bind Mounts Docs](https://docs.docker.com/storage/bind-mounts/)
* [Go Static Binaries Explained](https://www.arp242.net/static-go.html)
* [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*