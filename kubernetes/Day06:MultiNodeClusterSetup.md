# Kubernetes — Day 06

## 🎯 Why Local Installation Over Managed Services?

| Aspect | Managed (GKE/AKS/EKS) | Local (Kind) |
|---|---|---|
| Control plane access | Abstracted away | Fully exposed |
| Learning depth | Shallow — no troubleshooting reps | Deep — you break it, you fix it |
| Cost | Often billed | Free (just Docker) |
| CKA relevance | Low — exam expects raw cluster admin | High — mirrors exam environment |

> **Key takeaway:** Managed services are great for production, but they hide exactly the internals the CKA exam tests you on.

---

## 🧰 What is Kind?

**Kind** = Kubernetes in Docker. It spins up full Kubernetes clusters using **Docker containers as nodes**, instead of VMs or bare metal.

```
┌─────────────────────────────────────────┐
│              Host Machine               │
│                                         │
│   ┌───────────────┐   ┌───────────────┐ │
│   │ control-plane │   │    worker     │ │
│   │ (docker cntr) │   │ (docker cntr) │ │
│   └───────────────┘   └───────────────┘ │
│                                         │
│         All managed by Kind + Docker    │
└─────────────────────────────────────────┘
```

---

## 🧩 What is a Kubernetes Cluster?

A **Kubernetes cluster** is a group of machines (called **nodes**) that work together to run your applications.

> **Analogy:** Think of it as a factory — the cluster is the entire factory, the nodes are the workers inside it, and Kubernetes is the factory manager deciding which worker does each job.

```
                Kubernetes Cluster
      +-----------------------------------+
      |                                   |
      |  Control Plane                    |
      |  - API Server                     |
      |  - Scheduler                      |
      |  - Controller Manager             |
      |                                   |
      |-----------------------------------|
      |                                   |
      | Worker Node 1                     |
      |   ├── Pod                         |
      |   ├── Pod                         |
      |                                   |
      | Worker Node 2                     |
      |   ├── Pod                         |
      |   ├── Pod                         |
      |                                   |
      +-----------------------------------+
```

### What's Inside a Cluster?

```
Cluster
│
├── Control Plane
│     ├── API Server
│     ├── Scheduler
│     ├── etcd
│     └── Controller Manager
│
├── Worker Node 1
│      ├── Pod A
│      └── Pod B
│
├── Worker Node 2
│      ├── Pod C
│      └── Pod D
│
└── Networking
```

### Real-World Example — "Building Amazon"

All of these microservices would run inside **one** Kubernetes cluster:

```
Kubernetes Cluster

├── User Service
├── Product Service
├── Cart Service
├── Payment Service
├── Redis
└── PostgreSQL
```

### Why Not Just Use One Machine?

If a single machine maxes out at, say, 20 Pods, and the app grows — you add another machine, and Kubernetes **automatically spreads Pods across both**:

```
               Cluster

        +--------------------+
        | Control Plane      |
        +--------------------+
                 |
     -----------------------------
     |                           |
+-----------+              +-----------+
| Worker 1  |              | Worker 2  |
|           |              |           |
| Pod A     |              | Pod C     |
| Pod B     |              | Pod D     |
+-----------+              +-----------+
```

Even with multiple physical/virtual machines, Kubernetes treats them as **one logical cluster**.

### Kind Cluster vs. Minikube Cluster

| | **Kind** | **Minikube** |
|---|---|---|
| Nodes run as | Docker containers | Typically a single VM/node |
| Multi-node support | Yes, via config file | Limited/less common |
| Node representation | Each container = one K8s node | One node hosts control plane + worker + pods together |

```
# Kind (multi-container)
Docker
┌───────────────────────┐
│ kind-control-plane    │
├───────────────────────┤
│ kind-worker           │
├───────────────────────┤
│ kind-worker2          │
└───────────────────────┘

# Minikube (single-node)
┌──────────────────────────┐
│ minikube node            │
│                          │
│ Control Plane            │
│ Worker                   │
│ Pods                     │
└──────────────────────────┘
```

### Why Multiple Clusters?

Separate clusters are commonly used to **isolate environments** and reduce the risk of test changes leaking into production:

```
Development Cluster        Testing Cluster           Production Cluster
      │                          │                          │
      ├── Backend                ├── Backend                ├── Backend
      ├── Frontend               ├── Frontend               ├── Frontend
      └── Database               └── Database               └── Database
```

### Verifying Your Cluster

```bash
# List all Kind clusters
kind get clusters

# Inspect nodes in the current cluster
kubectl get nodes
```

### ⚠️ Cluster vs. Context — Don't Confuse These

| Term | Meaning |
|---|---|
| **Cluster** | The actual Kubernetes environment — the real servers or containers running Kubernetes |
| **Context** | A saved connection profile in `kubectl` config — defines which cluster to talk to, which user, and optionally which namespace |

> One cluster can have multiple contexts pointing to it (different users/namespaces), which is why `kubectl config get-contexts` and `use-context` (covered below) matter so much for the CKA exam.

---

## ✅ Prerequisites

- **Docker** installed and running
- **Go 1.16** mentioned as a dependency for Kind

## 📦 Installation

```bash
# macOS (Homebrew)
brew install kind
```

---

## 🚀 Cluster Creation

### Single Node Cluster

```bash
kind create cluster --image <version> --name <name>
```

### Multi-Node Cluster (via YAML config)

For multi-node setups, define nodes explicitly in a config file — roles include `control-plane` and `worker`.

```yaml
# kind-multi-node-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

```bash
kind create cluster --config kind-multi-node-config.yaml --name multi-node-cluster
```

---

## 🔧 kubectl — The Standard CLI

`kubectl` is the tool used to interact with **any** Kubernetes cluster (local or managed).

⚠️ **Version matching matters** — ensure `kubectl` version is compatible with your cluster's server version to avoid API mismatches.

```bash
kubectl version --client
kubectl version
```

---

## 🔀 Managing Multiple Clusters — Context Switching

When running multiple clusters (common during CKA practice), you need to switch contexts to make sure commands target the right cluster.

| Action | Command |
|---|---|
| List all contexts | `kubectl config get-contexts` |
| Switch context | `kubectl config use-context <context-name>` |

```bash
# List contexts
kubectl config get-contexts
```

```bash
# Switch to a specific context
kubectl config use-context <context-name>
```

> 🎯 **CKA Exam Tip:** The exam presents multiple clusters/contexts — always verify your current context before running any task. A correct answer in the wrong context scores zero.

---

## 📚 Allowed Resources During the CKA Exam

- `kubernetes.io/docs` ✅
- `kubernetes.io/blog` ✅

### 📝 The Cheat Sheet

The official Kubernetes docs include a **Cheat Sheet** page packed with commonly used commands.

> **Recommendation:** Get intimately familiar with the cheat sheet's *layout* (not just contents) before exam day — knowing *where* things are saves precious minutes under time pressure.

---

## 🗂️ Glossary

| Term | Definition |
|---|---|
| **Kind** | Kubernetes in Docker — runs K8s clusters using Docker containers as nodes |
| **Control Plane** | The cluster's management layer (API server, scheduler, etcd, controller-manager) |
| **Context** | A combination of cluster + user + namespace that `kubectl` uses to target commands |
| **kubectl** | CLI tool for interacting with Kubernetes clusters |
| **CKA** | Certified Kubernetes Administrator exam |

---

## 🔗 Cross-References
- Precedes upcoming CKA topics: cluster architecture, workloads, networking, troubleshooting
- Complements DevOps/AWS track: local Kind clusters are a low-cost sandbox before deploying to EKS/ECS-based infra