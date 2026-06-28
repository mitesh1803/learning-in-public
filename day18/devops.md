![Progress](https://img.shields.io/badge/Progress-18%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 18 — Docker Networking

## 📝 Topic: Container Communication, Isolation & Custom Bridge Networks
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 24, 2026

---

## 🎯 Learning Objectives
* Understand why Docker networking exists — communication and isolation.
* Know the three network types: Bridge, Host, and Overlay.
* Understand the default `docker0` bridge and its security limitations.
* Create custom bridge networks for proper container isolation.
* Attach containers to specific networks at runtime.
* Use key Docker networking commands: `ls`, `create`, `inspect`.

---

## 🌐 Part 1 — Why Docker Networking Exists

Containers are isolated processes. Without networking, they have no way to talk to each other, to the host, or to the outside world.

Docker networking solves two problems simultaneously:

```
Problem 1 — COMMUNICATION:
  Container A (backend API) needs to talk to Container B (database)
  Container C (frontend)    needs to talk to Container A (backend)
  All containers need internet access for package downloads
  
  → Docker networking provides the channels for all of this

Problem 2 — ISOLATION:
  Finance service container holds sensitive payment data
  Login service container handles user authentication
  
  These two must NOT be able to communicate with each other directly
  A compromised login container must not reach the finance container
  
  → Docker networking provides the walls between them
```

> **The goal:** Connect what should be connected. Separate what should be separated.

---

## 🔌 Part 2 — The Three Network Types

### 1️⃣ Bridge Networking (Default)

The network type Docker uses when no network is specified.

```
How it works:
  Docker creates a virtual ethernet bridge called docker0 on the host
  Every container gets a virtual network interface (veth)
  The veth connects the container to docker0
  docker0 connects to the host's physical network interface

Host machine
├── Physical NIC (eth0) → internet
└── docker0 (virtual bridge, 172.17.0.1)
    ├── Container A (172.17.0.2)
    ├── Container B (172.17.0.3)
    └── Container C (172.17.0.4)
```

**Default bridge behaviour:**

```bash
# When you run ANY container without specifying a network
docker run -d nginx
# → automatically placed on the default bridge (docker0)
# → gets IP in the 172.17.0.0/16 subnet
# → can reach other containers on the same bridge by IP
# → can reach the internet via the host's NAT
```

**The security problem with the default bridge:**

```
All containers on the default bridge can talk to each other
Login container → can reach Finance container → not acceptable
No isolation between services on the same default network
```

The default bridge is fine for learning. It is **not acceptable for production** when services should be isolated.

---

### 2️⃣ Host Networking

Removes all network isolation between the container and the host.

```
Normal (Bridge):
  Container → docker0 (NAT) → host NIC → internet
  Container has its own IP address (e.g. 172.17.0.2)
  Port mapping required: -p 8080:80

Host networking:
  Container → host NIC (directly)
  Container shares the host's IP address
  No port mapping needed — container ports = host ports directly
```

```bash
docker run -d --network host nginx
# Nginx now listens on the HOST's port 80 directly
# No -p flag needed — but also no isolation
```

**Why it is considered insecure:**

```
Container shares the host network stack completely
→ Container can see and bind to any port on the host
→ A compromised container can interfere with host networking
→ No isolation layer between container and host

Use case: when you need maximum network performance
          and isolation is not a concern (rare)
```

---

### 3️⃣ Overlay Networking

Used for **multi-host communication** — containers running on different physical machines or nodes.

```
Single Host (Bridge/Host is enough):
  Machine A: Container 1 ←→ Container 2

Multi-Host (needs Overlay):
  Machine A: Container 1  ←──[overlay network]──→  Container 2 :Machine B
  Machine A: Container 3  ←──[overlay network]──→  Container 4 :Machine C
```

**Where it's used:**

```
Kubernetes:     pod-to-pod networking across nodes uses overlay (Flannel, Calico, Weave)
Docker Swarm:   service-to-service communication across the swarm cluster
```

```bash
# Overlay requires Swarm mode or an external KV store
docker swarm init
docker network create --driver overlay my-overlay-network
```

> As a DevOps engineer, you won't typically configure overlay networks manually in modern environments — Kubernetes CNI plugins (Flannel, Calico) handle this. Understanding the concept is important for debugging and interviews.

---

## 🔒 Part 3 — Custom Bridge Networks: The Right Solution

### Why Custom Bridge Over Default Bridge

```
Default bridge (docker0):
  All containers can reach all other containers
  No isolation between services
  No DNS-based container name resolution (only IPs)

Custom bridge network:
  Only containers explicitly attached to it can communicate
  Complete isolation from containers on other custom networks
  ✅ DNS resolution by container name (no IP memorization)
```

### Creating a Custom Network

```bash
# Create a custom bridge network
docker network create finance-network
docker network create login-network

# Verify they were created
docker network ls
# NETWORK ID     NAME              DRIVER    SCOPE
# a1b2c3d4e5f6   bridge            bridge    local   ← default
# f6e5d4c3b2a1   finance-network   bridge    local   ← custom
# 123456789abc   login-network     bridge    local   ← custom
# 987654321def   host              host      local
# abc123def456   none              null      local
```

### Attaching Containers to Custom Networks

```bash
# Finance service — attached to finance-network only
docker run -d \
  --name payment-service \
  --network finance-network \
  payment-image:latest

docker run -d \
  --name database \
  --network finance-network \
  postgres:15

# Login service — attached to login-network only
docker run -d \
  --name auth-service \
  --network login-network \
  auth-image:latest
```

**Result:**

```
finance-network:              login-network:
  payment-service ✅            auth-service ✅
  database        ✅

payment-service → database:      ✅ can communicate (same network)
payment-service → auth-service:  ❌ cannot communicate (different network)
auth-service    → database:      ❌ cannot communicate (different network)
```

The finance containers are fully isolated from the login containers — even though they run on the same Docker host.

### DNS Resolution on Custom Networks

```bash
# On a custom bridge network, containers can reach each other by name
# No need to use IP addresses

docker run -d --name db --network finance-network postgres:15
docker run -d --name api --network finance-network my-api

# Inside the api container:
curl http://db:5432      # ✅ works — Docker resolves "db" to the container's IP
curl http://172.17.0.3   # also works, but fragile — IPs can change
```

> **This is a major advantage of custom networks.** Container IPs change when containers restart. Container names don't. DNS-based service discovery removes the need to hardcode IPs.

---

## 🔧 Part 4 — Key Networking Commands

### List All Networks

```bash
docker network ls
```

```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local    ← default bridge (docker0)
987654321def   host      host      local    ← host networking
abc123def456   none      null      local    ← no networking
f6e5d4c3b2a1   my-net    bridge    local    ← custom bridge
```

### Create a Network

```bash
# Basic creation (default bridge driver)
docker network create my-network

# With a custom subnet
docker network create \
  --driver bridge \
  --subnet 192.168.100.0/24 \
  --gateway 192.168.100.1 \
  my-custom-network
```

### Inspect a Network

```bash
docker network inspect finance-network
```

```json
[
  {
    "Name": "finance-network",
    "Driver": "bridge",
    "Subnet": "172.18.0.0/16",
    "Gateway": "172.18.0.1",
    "Containers": {
      "abc123...": {
        "Name": "payment-service",
        "IPv4Address": "172.18.0.2/16"
      },
      "def456...": {
        "Name": "database",
        "IPv4Address": "172.18.0.3/16"
      }
    }
  }
]
```

### Connect a Running Container to a Network

```bash
# Attach an already-running container to a second network
docker network connect login-network auth-service

# Now auth-service is on BOTH login-network AND the new network
# Useful when a container needs to bridge two isolated networks
```

### Disconnect and Remove

```bash
# Disconnect a container from a network
docker network disconnect finance-network payment-service

# Remove a network (all containers must be disconnected first)
docker network rm finance-network
```

### Run Container on a Specific Network

```bash
docker run -d \
  --name my-container \
  --network my-custom-network \
  -p 8080:80 \
  nginx
```

---

## 📊 Part 5 — Network Types Comparison

| Feature | Bridge (Default) | Bridge (Custom) | Host | Overlay |
|---|---|---|---|---|
| **Isolation** | ❌ All containers visible | ✅ Only attached containers | ❌ None | ✅ Per network |
| **DNS by name** | ❌ IP only | ✅ Container name works | N/A | ✅ Service name |
| **Performance** | Normal | Normal | Highest | Slight overhead |
| **Security** | Low | High | Lowest | High |
| **Multi-host** | ❌ | ❌ | ❌ | ✅ |
| **Use case** | Dev/learning | Production isolation | High-perf, no isolation | Kubernetes, Swarm |

---

## 🏗️ Part 6 — Real-World Architecture Example

A typical microservices setup with network isolation:

```
docker network create frontend-network
docker network create backend-network
docker network create db-network

# Frontend — public facing
docker run -d --name nginx-proxy \
  --network frontend-network \
  -p 80:80 nginx

# API — bridges frontend and backend
docker run -d --name api-server \
  --network frontend-network \
  my-api-image

docker network connect backend-network api-server
# api-server is now on BOTH frontend-network AND backend-network

# Auth service — backend only
docker run -d --name auth-service \
  --network backend-network \
  auth-image

# Database — completely isolated
docker run -d --name postgres-db \
  --network db-network \
  postgres:15

docker network connect db-network api-server
# api-server can now reach postgres-db

# Result:
# nginx-proxy ←→ api-server ✅   (frontend-network)
# api-server  ←→ auth-service ✅ (backend-network)
# api-server  ←→ postgres-db ✅  (db-network)
# nginx-proxy ← → auth-service ❌ (different networks)
# nginx-proxy ← → postgres-db ❌  (different networks)
# auth-service ← → postgres-db ❌ (different networks)
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Docker Networking** | The system that controls how containers communicate with each other and the outside world |
| **Bridge Network** | A virtual network using a software bridge (`docker0`) — the default Docker network type |
| **`docker0`** | The default virtual ethernet bridge Docker creates on the host |
| **veth** | Virtual ethernet pair — one end in the container, one end on the bridge |
| **Host Network** | Container shares the host's network stack directly — no isolation |
| **Overlay Network** | Multi-host network — connects containers across different machines (Kubernetes, Swarm) |
| **Custom Bridge** | A user-created bridge network — provides isolation and DNS resolution by container name |
| **Network Isolation** | Containers on different networks cannot communicate by default |
| **DNS Resolution** | On custom bridge networks, containers can reach each other by name, not just IP |
| **`docker network ls`** | Lists all Docker networks on the host |
| **`docker network create`** | Creates a new Docker network |
| **`docker network inspect`** | Shows detailed info about a network — containers, subnets, gateway |
| **`docker network connect`** | Attaches a running container to an additional network |
| **`docker network disconnect`** | Removes a container from a network |
| **`docker network rm`** | Deletes a network (must disconnect all containers first) |
| **`--network` flag** | Specifies which network a container joins at startup |
| **Subnet** | The IP address range for a network (e.g. `172.18.0.0/16`) |
| **Gateway** | The network's router IP — packets to other networks go through here |
| **NAT** | Network Address Translation — how containers reach the internet through the host |
| **CNI** | Container Network Interface — the Kubernetes standard for overlay networking plugins |

---

## 📂 Summary of Tasks
- ✅ Understood: Why Docker networking is needed — communication AND isolation.
- ✅ Understood: Bridge (default) — docker0 virtual bridge, all containers visible to each other.
- ✅ Understood: Host networking — no isolation, container shares host network stack.
- ✅ Understood: Overlay networking — multi-host, used by Kubernetes and Docker Swarm.
- ✅ Understood: The default bridge security problem — no isolation between services.
- ✅ Created: Custom bridge networks with `docker network create`.
- ✅ Attached: Containers to specific networks with `--network` flag.
- ✅ Used: `docker network ls`, `inspect`, `connect`, `disconnect`, `rm`.
- ✅ Understood: DNS resolution by container name on custom bridge networks.
- ✅ Designed: A multi-network microservices architecture with proper isolation.

---

## 💡 My Takeaway

The default bridge is a trap for production use. It works, it's convenient, and it looks fine — until you realize every container on it can reach every other container. For a simple learning project that's acceptable. For a system where a compromised login service could reach the payment database, it's a serious security gap.

Custom bridge networks solve both problems the video opened with — communication (containers on the same network can talk) and isolation (containers on different networks cannot). The DNS name resolution bonus is equally valuable: instead of hardcoding `172.18.0.3` in your app config (which changes every time a container restarts), you use the container name `database` and Docker resolves it automatically.

The multi-network pattern — where one container is connected to two networks to act as a bridge — is the real production pattern. Your API server needs to talk to the frontend network AND the backend network, but the frontend must never have a direct route to the database. Connecting one container to multiple networks with `docker network connect` is how that's achieved.

---

## 📈 Interview Questions:
In this video, *Abhishek* covers 12 scenario-based Docker interview questions and answers. Here are the questions discussed:

1. **What is Docker? / What is a container?** (1:21)
2. **How are containers different from virtual machines?** (2:36)
3. **What is Docker lifecycle?** (5:58)
4. **What are the different Docker components?** (8:35)
5. **What is the difference between Docker COPY and Docker ADD?** (11:43)
6. **What is the difference between CMD and Entrypoint in Docker?** (13:06)
7. **What are the networking types in Docker and what is the default?** (16:21)
8. **Can you explain how to isolate networking between containers?** (19:44)
9. **What is a multi-stage build in Docker?** (23:04)
10. **What are distroless images in Docker?** (26:47)
11. **What are some real-time challenges with Docker?** (30:33)
12. **What steps would you take to secure containers?** (34:36)
---

## 🔗 Resources
* [Abhishek Veermalla Docker Repo](https://github.com/iam-veeramalla/Docker-Zero-to-Hero)
* [Docker Networking Overview](https://docs.docker.com/network/)
* [Docker Bridge Network Docs](https://docs.docker.com/network/drivers/bridge/)
* [Docker Network Commands Reference](https://docs.docker.com/reference/cli/docker/network/)
* [Kubernetes CNI Plugins](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
