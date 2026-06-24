# Docker vs Podman 


## 1. Docker vs Podman — Feature Comparison

| Feature | Docker | Podman |
|---|---|---|
| **Architecture** | Client-server (daemon-based) | Daemonless |
| **Root requirement** | Requires root by default | Rootless by default |
| **Security** | Daemon runs as root (attack surface) | Each container runs as the invoking user |
| **Compatibility** | Industry standard | Docker CLI-compatible (`alias docker=podman`) |
| **Compose support** | `docker compose` (built-in) | `podman-compose` (separate install) |
| **Pod support** | No native pod concept | Native pods (like Kubernetes pods) |
| **Systemd integration** | Workaround needed | First-class (`podman generate systemd`) |
| **Image format** | OCI + Docker format | OCI-compliant |
| **Desktop GUI** | Docker Desktop (free for personal) | Podman Desktop (fully open source) |
| **Registry default** | Docker Hub | Configurable (Docker Hub, Quay, etc.) |
| **Kubernetes YAML** | `kompose` needed | `podman generate kube` built-in |

### When to choose Docker
- You're on a team where Docker is the standard
- You need Docker Desktop on Mac/Windows with smooth UX
- Your CI/CD pipelines are Docker-native (GitHub Actions, etc.)
- You rely heavily on Docker Compose for local dev

### When to choose Podman
- Security is a priority — rootless containers reduce risk
- You're targeting Kubernetes and want pod semantics locally
- You're on RHEL/Fedora/CentOS (Podman is the default there)
- You want systemd-managed containers on a Linux server
- You dislike having a background daemon consuming resources

### Key Mental Model

**Docker** = client talks to a `dockerd` daemon (runs as root) → daemon manages all containers.

**Podman** = each `podman` command forks the container directly as a child process, no daemon in between. Much closer to how a regular Linux process works.

---

## 2. How Docker Achieves (and Fails at) Security

### Linux Kernel Primitives Docker Uses

```
┌─────────────────────────────────────────────┐
│              Host Linux Kernel              │
│                                             │
│  ┌──────────────┐    ┌──────────────┐       │
│  │  Namespaces  │    │   Cgroups    │       │
│  │  (isolation) │    │  (resource   │       │
│  │              │    │   limits)    │       │
│  └──────────────┘    └──────────────┘       │
│                                             │
│  ┌──────────────┐    ┌──────────────┐       │
│  │  Seccomp     │    │ Capabilities │       │
│  │ (syscall     │    │  (privilege  │       │
│  │  filter)     │    │   control)   │       │
│  └──────────────┘    └──────────────┘       │
└─────────────────────────────────────────────┘
```

### Namespaces (Main Isolation Tool)

| Namespace | What it isolates |
|---|---|
| `pid` | Process tree — container can't see host PIDs |
| `net` | Network stack — separate interfaces, IPs |
| `mnt` | Filesystem mounts |
| `uts` | Hostname |
| `ipc` | Inter-process communication |
| `user` | User/UID mapping |

### cgroups
Prevents one container from eating all CPU/RAM and starving others. Resource isolation, not security isolation.

### Seccomp — Syscall Filtering
Docker applies a default seccomp profile that blocks ~44 dangerous syscalls (`ptrace`, `reboot`, `mount`, etc.). A compromised container can't call these.

### Linux Capabilities
Instead of full root, Docker drops most capabilities by default. A container only gets a minimal set like `CHOWN`, `NET_BIND_SERVICE` — not `SYS_ADMIN`, `SYS_PTRACE` etc.

---

## 3. Where Docker Security Breaks — Real Attack Vectors

### 🔴 Attack 1: Container Escape via Privileged Mode

```bash
# Developer runs this for convenience
docker run --privileged myapp

# Attacker inside container can now:
mkdir /mnt/host
mount /dev/sda1 /mnt/host   # Mount the HOST filesystem!
chroot /mnt/host             # Now you're on the host
# Game over — full host access
```

`--privileged` disables ALL security mechanisms. Very common mistake.

---

### 🔴 Attack 2: Docker Socket Mount

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock myapp
#                 ↑ This is catastrophic
```

If an attacker gets inside this container, they can:

```bash
# Inside the container
docker run -v /:/host --privileged ubuntu chroot /host
# Full host filesystem access — instant escape
```

The Docker socket = God mode. Never mount it unless absolutely necessary.

---

### 🔴 Attack 3: Running as Root Inside Container

```dockerfile
# Bad — default if you don't specify
FROM node:18
COPY . .
RUN npm install
CMD ["node", "server.js"]   # Runs as root!
```

If there's a vulnerability in your app, the attacker has root inside the container.

**Fix:**
```dockerfile
FROM node:18
COPY . .
RUN npm install
RUN useradd -m appuser
USER appuser          # Drop privileges
CMD ["node", "server.js"]
```

---

### 🔴 Attack 4: Shared Kernel = Kernel Exploits Escape Everything

This is the fundamental architectural limit. Namespaces isolate userspace. The kernel is shared.

```
Container A  →  exploits kernel CVE  →  gains host kernel access
                                      → can now touch Container B, C, D
```

A VM would stop this — each VM has its own kernel. Docker cannot.

---

## 4. Docker Defense Layers

```
1. Never use --privileged
2. Never mount Docker socket
3. Always USER non-root in Dockerfile
4. Use read-only filesystems  --read-only
5. Drop all capabilities, add only what's needed
   --cap-drop ALL --cap-add NET_BIND_SERVICE
6. Enable Seccomp + AppArmor profiles
7. Scan images (Trivy, Snyk)
8. Use network policies — containers shouldn't
   talk to each other unless explicitly allowed
```

### Network Isolation Between Containers

By default on the same Docker network, containers CAN reach each other:

```bash
# Fix: use separate networks + explicit allowlisting
docker network create frontend-net
docker network create backend-net

# Only connect services that need to talk
docker run --network frontend-net nginx
docker run --network backend-net postgres
```

---

## 5. Docker vs VM — Security Comparison

```
VMs:                          Docker:
┌──────────┬──────────┐       ┌──────────┬──────────┐
│  VM 1    │  VM 2    │       │  Cont 1  │  Cont 2  │
│  Kernel  │  Kernel  │       │          │          │
├──────────┴──────────┤       ├──────────┴──────────┤
│    Hypervisor       │       │    Docker Daemon     │
├─────────────────────┤       ├─────────────────────┤
│    Host Kernel      │       │    Host Kernel  ←── shared!
└─────────────────────┘       └─────────────────────┘

Kernel exploit in VM   → only that VM affected
Kernel exploit in container → ALL containers affected
```

---

## 6. How Podman Achieves Security

### Architecture Comparison

```
DOCKER:
  CLI          Daemon (root)      Containers
 docker  ───►  dockerd  ───►  [C1] [C2] [C3]
               (always            all owned
               running            by root)
               as ROOT

PODMAN:
  CLI              No Daemon         Containers
 podman  ───►  (fork-exec directly)  [C1] [C2] [C3]
  (as user)      conmon               all owned
                (monitor)            by invoking user
```

---

### Podman Security Feature 1: Rootless Containers

In Docker, even "non-root inside container" means the container process is still owned by root on the host.

In Podman, the entire container runs as **your user** on the host:

```bash
# Docker — host process ownership
$ ps aux | grep myapp
root     12345  ...  myapp      ← owned by ROOT on host

# Podman — host process ownership
$ ps aux | grep myapp
mitesh   12345  ...  myapp      ← owned by YOU on host
```

Even if an attacker escapes the container, they're just a regular user — not root.

---

### Podman Security Feature 2: User Namespace Remapping

```
Inside Container          Host OS
━━━━━━━━━━━━━━━━━━        ━━━━━━━━━━━━━━━━━━━━━━━
root (UID 0)      ──────► mitesh (UID 1000)
user1 (UID 1)     ──────► UID 100001
user2 (UID 2)     ──────► UID 100002
```

- Inside the container, the app thinks it's running as root (UID 0)
- On the host, that UID 0 maps to your unprivileged user
- The kernel enforces this mapping via `/etc/subuid` and `/etc/subgid`

```bash
# Your subuid range
cat /etc/subuid
mitesh:100000:65536
#      ↑start  ↑count — 65536 UIDs to map into containers
```

"Root inside container" is a fiction — it has zero host privileges.

---

### Podman Security Feature 3: No Daemon

```
Docker risk:
If dockerd is compromised → ALL containers compromised
If docker socket is exposed → instant root on host

Podman:
No daemon → nothing to compromise at the system level
No socket → no /var/run/docker.sock to accidentally mount
Each container is isolated at OS process level
```

Podman uses a small monitor process called `conmon` (container monitor) per container — handles stdio and exit codes only. Tiny, minimal, not privileged.

---

### Podman Security Feature 4: SELinux Integration (First-Class)

```
Without SELinux:
Container escapes namespace → can read host files

With SELinux + Podman:
Container escapes namespace → SELinux policy blocks file access
→ attacker hits a second wall
```

Every container gets a unique SELinux label. Even if two containers escape their namespaces, SELinux prevents them from accessing each other's files.

```bash
# Podman auto-applies SELinux labels
podman run -v /data:/data:Z myapp
#                         ↑ Z = relabel for SELinux automatically
```

Docker requires manual SELinux configuration. Podman does it by default.

---

### Podman Security Feature 5: Capabilities in Rootless Mode

```
Docker default caps:          Podman rootless caps:
✓ CHOWN                       ✓ CHOWN (only within user ns)
✓ NET_BIND_SERVICE            ✗ NET_BIND_SERVICE (ports < 1024 blocked)
✓ SYS_CHROOT                  ✗ SYS_CHROOT
✓ SETUID / SETGID             ✓ SETUID (only within user ns)
✗ SYS_ADMIN                   ✗ SYS_ADMIN
```

In rootless Podman, even granted capabilities only apply within the user namespace — no effect on the host.

---

### Cross-Container Breach Comparison

```
Scenario: Attacker compromises Container A

DOCKER:
Container A
  → exploits app vuln
  → gets shell inside container (as root)
  → finds docker.sock mounted (common mistake)
  → talks to dockerd
  → launches new container with -v /:/host
  → full host access
  → Container B, C, D — all readable

PODMAN:
Container A
  → exploits app vuln
  → gets shell inside container (as "root" = UID 0 inside)
  → no daemon, no socket to exploit
  → tries to read /etc/shadow → SELinux blocks it
  → tries to access Container B's files → different SELinux label, blocked
  → tries kernel exploit → still just regular user on host
  → blast radius: only what invoking user can do
```

---

## 7. What Podman Doesn't Fix

| Risk | Podman's stance |
|---|---|
| Kernel CVE | Still shared kernel — same fundamental limit as Docker |
| Misconfigured `--privileged` | Still dangerous, still possible |
| Malicious image | You still pull and run it — your problem |
| App-level vulns | Container security ≠ app security |

---

## 8. Summary

```
Docker security model:
"Trust the daemon, trust the socket, hope nobody misuses it"

Podman security model:
"Each container is just a process owned by you,
 with fake root inside, bounded by user namespaces,
 watched by SELinux, with no privileged daemon to exploit"
```

Podman shrinks the **blast radius**. A breach in one container stays contained to what your Linux user can do, not what root can do. That's a fundamentally different threat model.

---

*Notes compiled from a study session on container security — Docker vs Podman architecture, isolation mechanisms, attack vectors, and defense strategies.*