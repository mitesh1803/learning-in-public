![Progress](https://img.shields.io/badge/Progress-14%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 14 — Project Management Tools + Introduction to Containers

## 📝 Topic: Agile, Jira, ServiceNow & The Evolution from VMs to Docker Containers
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 19, 2026

---

## 🎯 Learning Objectives
* Understand Agile methodology — why Sprints replaced Waterfall for modern delivery.
* Know the key project management tools: Jira, Confluence, SharePoint, ServiceNow, GitHub Boards.
* Understand Incident Management and Change Management as formal processes in ServiceNow.
* Trace the evolution of infrastructure: Physical → Virtual Machines → Containers.
* Understand what a container is and why it solves the VM overhead problem.
* Know the Docker lifecycle: Dockerfile → Image → Container.
* Understand Docker's single point of failure problem and how Buildah addresses it.

---

## 🏢 Part 1 — Project Management for DevOps Engineers

> *"Beyond technical skills, understanding how an organization tracks its work is crucial for success in a new role."*

A DevOps engineer who can't read a Jira board, doesn't know how to raise a ServiceNow change request, or has never seen Confluence documentation will struggle in their first few weeks — regardless of how strong their technical skills are. These tools are the organizational nervous system.

---

## 🔄 Part 2 — Agile Methodology

### Waterfall vs Agile

```
WATERFALL (traditional):
  Requirements → Design → Development → Testing → Deployment
  ←────────────────────── 6-18 months ─────────────────────→

  Problems:
  → Customer sees the product only at the very end
  → Requirements change mid-way → expensive rework
  → One long delivery cycle → risky, slow feedback
  → If final product is wrong → entire effort wasted

AGILE:
  Sprint 1 (2 weeks): small working slice → ship → get feedback
  Sprint 2 (2 weeks): next slice → ship → get feedback
  Sprint 3 (2 weeks): next slice → ...
  ←── each sprint ──→←── each sprint ──→←── each sprint ──→

  Benefits:
  → Customer sees working software every 2 weeks
  → Feedback incorporated in the next sprint
  → Mistakes caught and corrected early
  → Risk is distributed across many small deliveries
```

### Key Agile Concepts

| Term | What it means |
|---|---|
| **Sprint** | A fixed time-box (usually 2 weeks) of focused development work |
| **Backlog** | The ordered list of all planned features, bugs, and tasks |
| **Sprint Planning** | Ceremony where team pulls items from backlog into the sprint |
| **Daily Standup** | 15-minute daily sync: what did I do, what will I do, any blockers? |
| **Sprint Review** | Demo of completed sprint work to stakeholders |
| **Retrospective** | Team reflection: what went well, what to improve |
| **Velocity** | Amount of work (story points) a team completes per sprint |
| **Epic** | A large body of work broken into multiple stories across sprints |
| **Story** | A user-facing feature or requirement — small enough to complete in one sprint |
| **Bug** | A defect in existing functionality — tracked and prioritized like stories |
| **Story Points** | Relative effort estimate — not hours, but complexity |

### Agile in a DevOps Context

```
Sprint Planning     → DevOps picks up infrastructure stories
During Sprint       → DevOps builds CI/CD pipelines, provisions infra
Sprint Review       → Demo working deployments, running pipelines
Retrospective       → Discuss: did deployments block the team? What to automate next?
```

---

## 📋 Part 3 — Jira

The industry-leading project tracking tool. Used to manage everything from new features to production bugs to DevOps infrastructure tasks.

### The Jira Work Item Hierarchy

```
Project
  └── Epic           (large goal, e.g., "Set up CI/CD pipeline")
        └── Story    (deliverable task, e.g., "Configure GitHub Actions workflow")
              └── Sub-task  (granular step, e.g., "Add unit test stage to workflow")
```

### How DevOps Engineers Use Jira

```
A typical DevOps sprint:

Story: Set up remote Terraform backend
  → Sub-task: Create S3 bucket for state storage
  → Sub-task: Create DynamoDB table for state locking
  → Sub-task: Configure backend.tf
  → Sub-task: Document setup in Confluence

Story: Write Ansible playbook for Nginx deployment
  → Sub-task: Write tasks/main.yml
  → Sub-task: Create nginx.conf.j2 template
  → Sub-task: Test on staging servers
```

### Jira Board Views

```
KANBAN VIEW:
┌────────────┬──────────────┬──────────────┬──────────────┐
│   To Do    │  In Progress │  In Review   │     Done     │
├────────────┼──────────────┼──────────────┼──────────────┤
│ Story A    │ Story C      │ Story E      │ Story G      │
│ Story B    │ Story D      │              │ Story H      │
└────────────┴──────────────┴──────────────┴──────────────┘
```

**Getting started:** Jira offers a free trial — create an account, create a project, add stories, drag them through the board. Familiarity with the interface before starting a new job is a meaningful advantage.

---

## 📚 Part 4 — Confluence & SharePoint

### Confluence

A wiki-style knowledge base — where teams document technical decisions, runbooks, architecture diagrams, and onboarding guides.

```
Typical Confluence spaces for a DevOps team:
  /devops/runbooks/        → How to restart services, handle alerts
  /devops/architecture/    → System diagrams, network topology
  /devops/onboarding/      → New engineer setup guide
  /devops/incidents/       → Post-incident reports
  /devops/terraform/       → Module documentation
```

> **Why this matters for new joiners:** The fastest way to get productive is good documentation. The fastest way to frustrate a team is by asking questions that are already answered in Confluence.

### SharePoint

Microsoft's equivalent — used heavily in enterprises running Microsoft 365. Same concept: shared knowledge base, internal wikis, document storage.

```
SharePoint vs Confluence:
  Both are internal wikis
  SharePoint: Microsoft ecosystem, integrates with Teams/Office 365
  Confluence: Atlassian ecosystem, integrates with Jira
  
Choice depends on what the company uses for source control and ticketing.
```

---

## 🚨 Part 5 — ServiceNow

ServiceNow is the enterprise standard for two critical DevOps processes: **Incident Management** and **Change Management**.

### Incident Management — Responding to System Failures

```
Scenario:
  3:42 AM — CloudWatch alarm fires
  → CPU on prod database at 98% for 10 minutes
  → App returning 500 errors to users

ServiceNow Incident workflow:
  1. Alert creates an Incident automatically (INC0043821)
  2. On-call engineer is paged
  3. Engineer logs all actions in the Incident ticket
  4. Root cause identified → fix applied
  5. Incident marked Resolved
  6. Post-incident review written and attached
```

**Why the ticket matters:** A formal record of every production issue — what happened, when, who responded, what was done. Essential for:
* Regulatory compliance (finance, healthcare)
* Pattern analysis ("this is the third DB CPU incident this month")
* SLA tracking ("we resolved 95% of P1 incidents within 1 hour")

### Incident Priority Levels

| Priority | Description | Response Target |
|---|---|---|
| **P1 / Critical** | Production completely down, all users affected | Immediate response, 15-min updates |
| **P2 / High** | Major feature broken, significant user impact | Response within 1 hour |
| **P3 / Medium** | Minor degradation, some users affected | Response within 4 hours |
| **P4 / Low** | Cosmetic issue, workaround available | Next business day |

### Change Management — Planned Production Updates

Before any DevOps engineer can make a change to a production system in a regulated enterprise, a **Change Request** must be approved.

```
DevOps Engineer wants to: update the Kubernetes version in production

Change Management process:
  1. Engineer creates Change Request (CHG0012345) in ServiceNow
     → What: Upgrade K8s from 1.27 to 1.28
     → Why: Security patches, feature requirements
     → Risk: Medium — potential pod restarts
     → Rollback plan: Downgrade via Terraform
     → Maintenance window: Saturday 02:00-04:00 UTC
  
  2. Change Advisory Board (CAB) reviews the request
  3. Manager/architect approves
  4. Change is implemented in the approved window
  5. Change marked Successful or rolled back
```

**Why Change Management exists:** Uncoordinated changes to production are the #1 cause of outages. Two teams deploying to the same service at the same time, someone changing a firewall rule without telling the network team, a database schema migration during peak hours — Change Management prevents all of these.

```
Standard Change: Pre-approved, low-risk, repeated action (e.g., routine patching)
Normal Change:   Full CAB review required
Emergency Change: Production is broken, needs approval but on accelerated timeline
```

---

## 🔖 Part 6 — Read the Docs & GitHub/Azure Boards

### Read the Docs

```
Used by: Open source projects (Kubernetes, Ansible, Terraform docs)
Format:  ReStructuredText (.rst) or Markdown
Build:   Sphinx or MkDocs
Host:    readthedocs.io (free for open source)
```

The public-facing documentation for most major DevOps tools — when you visit `docs.ansible.com` or `terraform.io/docs`, that's often powered by Read the Docs infrastructure.

### GitHub Projects / Azure Boards

```
GitHub Projects:
  → Project boards built into GitHub
  → Issues become cards on the board
  → Ideal for: development teams, open source projects
  → Advantage: documentation, code, and tasks in one platform

Azure Boards:
  → Microsoft's Agile tracking tool
  → Part of Azure DevOps suite
  → Integrates with Azure Repos, Azure Pipelines, Azure Test Plans
  → Advantage: native integration with Microsoft CI/CD stack
```

**The trend:** GitHub Projects has become sophisticated enough to replace Jira for many development teams — especially those already on GitHub. The advantage is having issues, PRs, and project boards in a single platform with no integration overhead.

---

## 🐋 Part 7 — The Evolution to Containers

### Generation 1: Physical Servers

```
Physical Server Setup:
  1 server → 1 application
  Application uses: 2 cores, 8GB RAM
  Server has:       32 cores, 256GB RAM
  Waste:            30 cores, 248GB RAM ← idle 24/7

Adding more apps:
  App 1 uses MySQL 5.7
  App 2 needs MySQL 8.0
  Can't run both on one server without conflict
  → Buy another physical server for App 2
  → More waste
```

---

### Generation 2: Virtual Machines

VMs solved the single-server-per-application problem.

```
Physical Server (32 cores, 256GB RAM)
       ↓ Hypervisor creates:
  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
  │  VM 1   │  │  VM 2   │  │  VM 3   │  │  VM 4   │
  │ 8 cores │  │ 8 cores │  │ 8 cores │  │ 8 cores │
  │ 64GB    │  │ 64GB    │  │ 64GB    │  │ 64GB    │
  │ Ubuntu  │  │ CentOS  │  │ Ubuntu  │  │ Debian  │
  │         │  │         │  │         │  │         │
  │ App 1   │  │ App 2   │  │ App 3   │  │ App 4   │
  └─────────┘  └─────────┘  └─────────┘  └─────────┘
```

Better utilization. But a new problem emerged:

```
Each VM needs a full Operating System:
  Ubuntu OS:       ~4GB RAM, ~2 CPU cores
  App inside VM:   ~1GB RAM, ~0.5 CPU cores
  
  70-80% of the VM's resources are consumed by
  the OS itself — just to run one small application.
```

**The VM overhead problem:** You're running 4 full Linux kernels just to host 4 small applications. That's significant waste.

---

### Generation 3: Containers

Containers share the host OS kernel. There's no separate OS per application.

```
Physical Server (32 cores, 256GB RAM)
       ↓ Container Runtime (Docker/containerd)
  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐
  │Container 1│  │Container 2│  │Container 3│  │Container 4│
  │ App 1     │  │ App 2     │  │ App 3     │  │ App 4     │
  │ libs      │  │ libs      │  │ libs      │  │ libs      │
  │ ~200MB    │  │ ~150MB    │  │ ~300MB    │  │ ~180MB    │
  └───────────┘  └───────────┘  └───────────┘  └───────────┘
            ↕           ↕             ↕              ↕
  ┌────────────────────────────────────────────────────────┐
  │           Shared Host OS Kernel                        │
  └────────────────────────────────────────────────────────┘

No duplicate OS overhead per container
```

**Containers vs VMs:**

| Aspect | Virtual Machines | Containers |
|---|---|---|
| **OS** | Full OS per VM | Shared host kernel |
| **Size** | GBs (includes OS) | MBs (app + libs only) |
| **Startup** | Minutes (boot OS) | Seconds (start process) |
| **Isolation** | Hardware-level (hypervisor) | OS-level (namespaces/cgroups) |
| **Overhead** | High — OS consumes significant resources | Low — minimal overhead |
| **Portability** | Heavy — large VM images | Lightweight — small container images |

> **Containers provide logical isolation without the overhead of a full OS.** This is what makes them lightweight and easy to ship.

---

## 🐳 Part 8 — Docker Basics

### What a Container Bundles

```
Container = Application + Required Libraries + Minimal Dependencies

NOT included:
  ❌ Full OS kernel (shared from host)
  ❌ Device drivers
  ❌ System daemons unrelated to the app
```

### The Docker Lifecycle

```
Dockerfile  →  Docker Image  →  Running Container
(blueprint)    (snapshot)       (instance)

Like:
  Blueprint   →  House plan   →  Actual built house
  Class       →  Class def    →  Object instance
```

**Step by step:**

```bash
# Step 1: Write a Dockerfile (the blueprint)
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "start"]
```

```bash
# Step 2: Build an image from the Dockerfile
docker build -t my-nodejs-app:v1.0 .

# Lists your images
docker images
# REPOSITORY         TAG    IMAGE ID       SIZE
# my-nodejs-app      v1.0   a3f8c21b...    145MB
```

```bash
# Step 3: Run a container from the image
docker run -d -p 3000:3000 --name my-app my-nodejs-app:v1.0

# Check running containers
docker ps
# CONTAINER ID   IMAGE              STATUS         PORTS
# b7e4c12a...    my-nodejs-app:v1.0  Up 2 minutes  0.0.0.0:3000->3000/tcp
```

```bash
# Step 4: Stop and remove the container
docker stop my-app
docker rm my-app
```

### Docker Layer Caching

Docker builds images in **layers** — each instruction in the Dockerfile adds a layer.

```
FROM node:20-alpine          # Layer 1: base image
WORKDIR /app                 # Layer 2: set working dir
COPY package*.json ./        # Layer 3: copy package files
RUN npm install              # Layer 4: install dependencies (cached if package.json unchanged)
COPY . .                     # Layer 5: copy source code
CMD ["npm", "run", "start"]  # Layer 6: define startup command
```

**Why order matters:** Layers are cached. If Layer 4 is cached (node_modules), Docker skips re-installing dependencies on every build unless `package.json` changed. Always copy `package*.json` and run `npm install` BEFORE copying source code — source code changes constantly, dependencies change rarely.

---

## ⚠️ Part 9 — Docker's Limitation: Single Point of Failure

```
Docker Architecture:
  Docker CLI
       ↓
  Docker Daemon (dockerd)   ← the central engine
       ↓
  Container 1 | Container 2 | Container 3 | Container 4
```

**The problem:** Every container on the host depends on `dockerd`. If the Docker daemon crashes, stops, or hangs:

```
dockerd crashes
  → ALL containers on that host are affected
  → No container can be started, stopped, or managed
  → Single point of failure for the entire container infrastructure
```

Additionally, Docker builds require the daemon running — developers need root/sudo access to it, which is a security concern.

---

## 🔨 Part 10 — Buildah: The Alternative

Buildah addresses both Docker daemon problems:

```
Docker build:          Requires dockerd running (daemon dependency)
Buildah build:         Daemonless — builds OCI-compliant images without a daemon

Docker security:       Daemon runs as root — security risk
Buildah security:      Runs as non-root user — better security posture

Docker images:         Layered images (complexity, larger size potential)
Buildah images:        More control over layers — can reduce complexity
```

```bash
# Build an image with Buildah (no Docker daemon needed)
buildah bud -t my-nodejs-app:v1.0 .

# Push to a registry
buildah push my-nodejs-app:v1.0 docker://registry.example.com/my-nodejs-app:v1.0
```

**Buildah + Podman** is increasingly the enterprise alternative to Docker — especially in environments where running a privileged daemon is a security concern (Kubernetes pods, CI/CD environments).

---

## 📊 Part 11 — Infrastructure Evolution Summary

```
ERA             TECHNOLOGY     ISOLATION      OVERHEAD      STARTUP
─────────────────────────────────────────────────────────────────────
Pre-2000s       Physical       Dedicated      None/full     N/A
                Servers        hardware       machine       (physical)

2000s-2010s     Virtual        Hypervisor     Full OS       2-5 minutes
                Machines       level          per VM

2010s-now       Containers     OS namespace   App + libs    Seconds
                               level          only
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Agile** | Iterative delivery methodology — work in short sprints rather than long waterfall phases |
| **Sprint** | A fixed 2-week cycle of planned, deliverable work |
| **Jira** | Atlassian's project tracking platform — the industry standard for Agile teams |
| **Confluence** | Atlassian's wiki and documentation platform |
| **SharePoint** | Microsoft's enterprise knowledge-sharing and document storage platform |
| **ServiceNow** | Enterprise ITSM platform — manages Incidents, Changes, Problems |
| **Incident** | An unplanned disruption to a production system requiring response |
| **Change Request** | A formal approval request to modify production systems |
| **CAB** | Change Advisory Board — reviews and approves normal change requests |
| **P1/Critical** | The highest severity incident — full production outage |
| **Container** | A lightweight, isolated runtime environment that shares the host OS kernel |
| **VM** | A full virtual computer with its own OS, running on a hypervisor |
| **Dockerfile** | A script defining how to build a Docker image |
| **Docker Image** | A read-only snapshot of a container — the blueprint |
| **Docker Container** | A running instance of a Docker image |
| **Layer** | A cached instruction step in a Docker image build |
| **Docker Daemon (`dockerd`)** | The central Docker engine — runs as root, manages all containers |
| **Single Point of Failure** | A component whose failure takes down the entire system |
| **Buildah** | A daemonless, rootless OCI image builder — alternative to Docker build |
| **OCI** | Open Container Initiative — the standard format for container images |
| **Podman** | A daemonless container runtime — Docker-compatible alternative |
| **Namespace** | Linux kernel feature that provides process isolation for containers |
| **cgroups** | Linux kernel feature that limits resource usage (CPU, memory) per container |
| **Read the Docs** | Documentation hosting platform — used by most major open source projects |
| **GitHub Projects** | GitHub's built-in project management boards — tracks Issues as tasks |
| **Azure Boards** | Microsoft's Agile tracking tool within the Azure DevOps suite |

---

## 📂 Summary of Tasks
- ✅ Understood: Agile vs Waterfall — why iterative delivery in Sprints beats long release cycles.
- ✅ Learned: Jira hierarchy — Project → Epic → Story → Sub-task, and how DevOps uses it.
- ✅ Understood: Confluence and SharePoint as internal knowledge bases.
- ✅ Understood: ServiceNow Incident Management — P1-P4 severity, response workflow.
- ✅ Understood: ServiceNow Change Management — Change Requests, CAB approval, maintenance windows.
- ✅ Reviewed: Read the Docs, GitHub Projects, and Azure Boards as alternatives.
- ✅ Traced: Infrastructure evolution — Physical → VMs → Containers and why each transition happened.
- ✅ Understood: What a container is — app + libs + shared kernel, no full OS overhead.
- ✅ Learned: Docker lifecycle — Dockerfile → Image → Container with real commands.
- ✅ Understood: Docker layer caching and why instruction order matters.
- ✅ Understood: Docker's single point of failure — the daemon dependency problem.
- ✅ Understood: Buildah as a daemonless, rootless alternative to Docker build.

---

## 💡 My Takeaway

The ServiceNow section changed something for me. I'd thought of Change Management as bureaucracy — approvals slowing down delivery. But the framing of "uncoordinated changes are the #1 cause of production outages" recontextualized it. The change request isn't a blocker; it's coordination infrastructure. Two teams touching the same service without knowing it is a real, common outage cause. The paperwork prevents the post-mortem.

On containers: the insight that clicked wasn't "containers are lighter than VMs" (I'd heard that) — it was understanding *why*. A VM runs a full OS for each application because the hypervisor provides hardware-level isolation and each instance needs its own kernel. A container gets isolation from Linux namespaces and cgroups without needing its own kernel. That's why it's megabytes instead of gigabytes and seconds instead of minutes — there's no OS to boot.

---

## 📈 Next Up
**Day 21:** Docker hands-on — writing Dockerfiles, building images, running containers, Docker Compose for multi-service apps, and pushing images to Docker Hub.

---

## 🔗 Resources
* [Jira Free Account Setup](https://www.atlassian.com/software/jira/free)
* [Confluence Free Account](https://www.atlassian.com/software/confluence/free)
* [ServiceNow — What is ITSM?](https://www.servicenow.com/products/itsm.html)
* [Docker Documentation](https://docs.docker.com/)
* [Buildah Documentation](https://buildah.io/)
* [OCI Specification](https://opencontainers.org/)
* [Linux Namespaces Explained](https://www.man7.org/linux/man-pages/man7/namespaces.7.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
