![Progress](https://img.shields.io/badge/Progress-20%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 20 — Kubernetes in Production + Pods Deep Dive

## 📝 Topic: K8s Distributions, KOPS Cluster Management & Your First Pod
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 26, 2026

---

## 🎯 Learning Objectives
* Distinguish development Kubernetes tools from production-grade distributions.
* Know the major Kubernetes distributions and when each is used.
* Understand why managed services (EKS/AKS/GKE) exist alongside self-managed clusters.
* Use KOPS to manage the full cluster lifecycle on AWS.
* Define what a Pod is and why Kubernetes uses Pods instead of raw containers.
* Write a `pod.yaml` manifest and deploy it with `kubectl`.
* Debug a running Pod using `describe` and `logs`.
* Set up a local cluster with Minikube for development.

---

## 🏗️ Part 1 — Development vs Production Kubernetes

### Development Tools — Great for Learning, Not for Production

```
Tool        Use Case                          What's Missing
──────────────────────────────────────────────────────────────
Minikube    Local single-node cluster         No HA, single node only
k3s         Lightweight, edge/IoT             Stripped down, not full K8s
k3d         k3s in Docker                     Learning/CI only
MicroK8s    Snap-installable K8s              No HA control plane
```

All of these are excellent for learning and local development. None of them are suitable for production because they lack **high availability** — one node goes down, the cluster goes down.

### Why Production Needs a Different Approach

```
Production requirements that dev tools don't provide:
  → High Availability (HA) control plane — multiple API server replicas
  → etcd cluster with replication (not single-node etcd)
  → Automatic security patching and CVE response
  → Enterprise support with SLAs
  → Compliance and audit logging
  → Automatic upgrades with zero downtime
```

---

## 🌐 Part 2 — Kubernetes Distributions

A Kubernetes distribution packages the core Kubernetes engine with additional tooling, security defaults, support, and sometimes a UI.

### The Major Distributions

| Distribution | Provider | Best For |
|---|---|---|
| **Vanilla Kubernetes** | Open source | Staging/pre-prod, cost saving, full control |
| **OpenShift** | Red Hat | Enterprises needing integrated security, CI/CD, registry |
| **Rancher** | SUSE | Multi-cluster management, on-premise + cloud |
| **VMware Tanzu** | VMware | Enterprises in VMware ecosystem |
| **EKS** | AWS | Production on AWS — managed control plane |
| **AKS** | Azure | Production on Azure — managed control plane |
| **GKE** | Google Cloud | Production on GCP — most mature managed K8s |

### Self-Managed vs Managed Services

```
Self-Managed (Vanilla K8s / OpenShift on-prem):
  ✅ Full control over every component
  ✅ Lower cost per cluster
  ❌ You manage upgrades, HA, etcd backups
  ❌ You're responsible when things break
  → Best for: dev/test/staging environments, cost-sensitive workloads

Managed (EKS / AKS / GKE):
  ✅ Cloud provider manages the control plane
  ✅ Automatic upgrades, patching, HA built in
  ✅ Enterprise support included
  ❌ Higher cost
  ❌ Less control over control plane
  → Best for: production workloads, regulated industries
```

**Real-world cost strategy:**

```
Development clusters (100s of engineers):  self-managed vanilla K8s
                                           → save on managed service costs

Production clusters (customer-facing):     EKS / AKS / GKE
                                           → pay for managed reliability and support
```

---

## ⚙️ Part 3 — KOPS: Kubernetes Operations

**KOPS** is the primary tool for managing the **full lifecycle** of self-managed Kubernetes clusters — creation, upgrades, configuration changes, and deletion — especially on AWS.

```
KOPS capabilities:
  → Create clusters         (kops create cluster)
  → Upgrade clusters        (kops upgrade cluster)
  → Modify cluster config   (kops edit cluster)
  → Rolling updates         (kops update cluster)
  → Delete clusters         (kops delete cluster)
  → Manage hundreds of clusters using S3 as a state store
```

### Prerequisites

```bash
# 1. Python 3
python3 --version

# 2. AWS CLI
aws --version

# 3. kubectl
kubectl version --client

# 4. Install KOPS
curl -Lo kops https://github.com/kubernetes/kops/releases/download/v1.28.0/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/
kops version
```

### AWS Configuration

```bash
# Configure AWS credentials
aws configure
# AWS Access Key ID:     <your-access-key>
# AWS Secret Access Key: <your-secret-key>
# Default region:        us-east-1
# Default output format: json
```

> ⚠️ **Always use an IAM user, never the root account.** The IAM user needs permissions for EC2, S3, Route53, IAM, and VPC.

### State Storage — S3 Bucket

KOPS stores all cluster configuration in an **S3 bucket**. This is what allows KOPS to manage hundreds of clusters — the state is centralized and versioned.

```bash
# Create S3 bucket for KOPS state
aws s3 mb s3://my-kops-state-store --region us-east-1

# Enable versioning (recommended — allows rollback of cluster config)
aws s3api put-bucket-versioning \
  --bucket my-kops-state-store \
  --versioning-configuration Status=Enabled

# Export for KOPS to use
export KOPS_STATE_STORE=s3://my-kops-state-store
```

### Creating a Cluster

```bash
# For testing — use .k8s.local (no Route53 domain needed)
kops create cluster \
  --name=mycluster.k8s.local \
  --state=s3://my-kops-state-store \
  --zones=us-east-1a \
  --node-count=2 \
  --node-size=t3.medium \
  --master-size=t3.medium \
  --dns-zone=mycluster.k8s.local

# Review what will be created
kops update cluster --name mycluster.k8s.local --state=s3://my-kops-state-store

# Actually create it (adds --yes)
kops update cluster --name mycluster.k8s.local --state=s3://my-kops-state-store --yes

# Validate cluster is ready (takes 5-10 mins)
kops validate cluster --name mycluster.k8s.local --state=s3://my-kops-state-store
```

### Domain Management

| Environment | DNS Approach |
|---|---|
| **Testing** | Use `.k8s.local` suffix — no real domain needed |
| **Production** | Register a domain → configure Route53 hosted zone → KOPS manages DNS records |

### ⚠️ Cost Warning

```
KOPS creates real AWS resources that COST MONEY:
  EC2 instances    → master + worker nodes
  EBS volumes      → node storage
  S3 bucket        → state storage
  Route53          → DNS (if using a domain)
  ELB              → load balancer (if configured)

Always run kops delete cluster when done learning:
  kops delete cluster --name mycluster.k8s.local --state=s3://my-kops-state-store --yes
```

---

## 🫛 Part 4 — Introduction to Pods

### What is a Pod?

> *"A Pod is the smallest deployable unit in Kubernetes. It acts as a wrapper around one or more containers, defining how they should run."*

```
Docker world:         docker run nginx
                      → imperative, one-off command, no record

Kubernetes world:     pod.yaml → kubectl apply
                      → declarative, version-controlled, reproducible
```

### Why Pods Instead of Containers Directly?

```
Problem with raw containers in K8s:
  Docker: docker run -p 8080:80 -v /data:/app -e ENV=prod nginx
          → long command, hard to version, hard to review, easy to get wrong

Kubernetes Pod:
  pod.yaml → stored in Git → reviewed in PR → applied consistently
           → anyone on the team runs the same thing
           → K8s uses the YAML as the source of truth
```

A Pod provides Kubernetes with a **declarative specification** of what to run and how to run it.

### Single vs Multi-Container Pods

```
Single-container Pod (most common):
  ┌────────────────────────────┐
  │          Pod               │
  │   ┌──────────────────┐    │
  │   │  nginx container  │    │
  │   └──────────────────┘    │
  │   IP: 10.244.0.5          │
  └────────────────────────────┘

Multi-container Pod (sidecar pattern):
  ┌──────────────────────────────────────────┐
  │                  Pod                     │
  │   ┌─────────────┐  ┌──────────────────┐ │
  │   │   App       │  │  Log Collector   │ │
  │   │  container  │  │  (sidecar)       │ │
  │   └─────────────┘  └──────────────────┘ │
  │   Shared network (localhost)             │
  │   Shared storage (same volumes)          │
  │   Single IP: 10.244.0.5                 │
  └──────────────────────────────────────────┘
```

**Why group containers in one Pod?**

```
Containers in the same Pod:
  → Share the same network namespace (communicate via localhost)
  → Share the same storage volumes
  → Are always scheduled on the same node together
  → Live and die together — scaled together as a unit

Use case: App container + sidecar that ships its logs to a central logging system
          Both need to be co-located and share the log file volume
```

### IP Addressing in Pods

```
Kubernetes assigns ONE IP address per Pod — not per container.

Pod IP: 10.244.0.5

Container A inside Pod → accessible at 10.244.0.5:8080
Container B inside Pod → accessible at 10.244.0.5:9090
Container A → Container B → use localhost:9090 (same network namespace)
```

---

## 🖥️ Part 5 — Local Setup with Minikube

```bash
# Start a local single-node cluster
minikube start

# Verify the cluster is running
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   2m    v1.28.0

# SSH into the Minikube node (useful for debugging)
minikube ssh

# Stop the cluster
minikube stop

# Delete the cluster entirely
minikube delete
```

### kubectl — The Cluster CLI

```
kubectl is to Kubernetes what docker is to Docker.
  docker run  → kubectl apply -f
  docker ps   → kubectl get pods
  docker logs → kubectl logs
  docker exec → kubectl exec
```

---

## 📝 Part 6 — Writing Your First Pod YAML

### The pod.yaml File

```yaml
# pod.yaml
apiVersion: v1           # API version for Pod resources
kind: Pod                # the resource type
metadata:
  name: nginx-pod        # name of the pod
  labels:
    app: nginx           # labels for grouping and selecting
    environment: dev
spec:
  containers:
    - name: nginx        # container name
      image: nginx:latest  # Docker image to use
      ports:
        - containerPort: 80   # port the container listens on
```

### Multi-Container Pod Example

```yaml
# multi-container-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
    - name: app
      image: my-app:latest
      ports:
        - containerPort: 8080
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app

    - name: log-collector
      image: fluent/fluent-bit:latest
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/app   # same volume, different container

  volumes:
    - name: shared-logs
      emptyDir: {}    # temporary shared storage, lives with the Pod
```

---

## ⚡ Part 7 — Deploying and Managing Pods

### Deploy a Pod

```bash
# Create the pod from the YAML file
kubectl create -f pod.yaml

# Or use apply (create if not exists, update if exists)
kubectl apply -f pod.yaml
```

### Verify the Pod is Running

```bash
kubectl get pods
# NAME        READY   STATUS    RESTARTS   AGE
# nginx-pod   1/1     Running   0          30s

# Extended view — shows node and cluster IP
kubectl get pods -o wide
# NAME        READY   STATUS    RESTARTS   AGE   IP            NODE
# nginx-pod   1/1     Running   0          30s   10.244.0.5    minikube
```

### Delete a Pod

```bash
kubectl delete pod nginx-pod

# Or delete using the YAML file
kubectl delete -f pod.yaml
```

---

## 🐛 Part 8 — Debugging Pods

### `kubectl describe pod` — Full Status Report

```bash
kubectl describe pod nginx-pod
```

```
Name:         nginx-pod
Namespace:    default
Node:         minikube/192.168.49.2
Start Time:   Mon, 26 Jun 2026 09:15:00 +0000
Labels:       app=nginx
Status:       Running
IP:           10.244.0.5

Containers:
  nginx:
    Image:        nginx:latest
    Port:         80/TCP
    State:        Running
      Started:    Mon, 26 Jun 2026 09:15:05 +0000
    Ready:        True

Events:
  Type    Reason     Age   Message
  ----    ------     ----  -------
  Normal  Scheduled  2m    Successfully assigned default/nginx-pod to minikube
  Normal  Pulling    2m    Pulling image "nginx:latest"
  Normal  Pulled     2m    Successfully pulled image "nginx:latest"
  Normal  Created    2m    Created container nginx
  Normal  Started    2m    Started container nginx
```

> **The Events section is the most useful part for debugging.** Failed pulls, OOM kills, scheduling failures — all appear here.

### `kubectl logs` — Container Output

```bash
# View logs of the main container
kubectl logs nginx-pod

# Follow logs in real time (like tail -f)
kubectl logs nginx-pod -f

# View logs of a specific container in a multi-container pod
kubectl logs nginx-pod -c log-collector

# View previous container's logs (if it crashed and restarted)
kubectl logs nginx-pod --previous
```

### `kubectl exec` — Run Commands Inside a Pod

```bash
# Open a shell inside the pod
kubectl exec -it nginx-pod -- /bin/bash

# Run a single command
kubectl exec nginx-pod -- cat /etc/nginx/nginx.conf
```

---

## ⚠️ Part 9 — Why Pods Alone Are Not Enough in Production

```
Problem 1 — No auto-healing:
  kubectl delete pod nginx-pod
  → Pod is gone. Nothing recreates it.
  → Service is down until manually redeployed.

Problem 2 — No scaling:
  Need 5 replicas of nginx?
  → Manually write 5 pod.yaml files? That's not scalable.

Problem 3 — No rolling updates:
  Update nginx from v1.24 to v1.25?
  → Delete all pods, create new ones → downtime.

Solution: Deployments
  → Wraps Pods with a ReplicaSet
  → Provides auto-healing (desired replicas always maintained)
  → Provides declarative rolling updates
  → Provides scaling with one command
```

> **Pods are the foundation. Deployments are what you actually use in production.**

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Minikube** | A local single-node Kubernetes cluster for development and learning |
| **k3s** | Lightweight K8s distribution for edge/IoT — not production-grade |
| **Vanilla Kubernetes** | Upstream Kubernetes without additional packaging — full control |
| **OpenShift** | Red Hat's enterprise Kubernetes distribution with integrated DevOps tooling |
| **EKS** | Elastic Kubernetes Service — AWS-managed Kubernetes control plane |
| **AKS** | Azure Kubernetes Service — Azure-managed Kubernetes |
| **GKE** | Google Kubernetes Engine — Google's managed Kubernetes service |
| **KOPS** | Kubernetes Operations — manages cluster lifecycle on AWS |
| **High Availability (HA)** | Multiple replicas of control plane components — no single point of failure |
| **Pod** | The smallest deployable unit in Kubernetes — wraps one or more containers |
| **Sidecar** | A secondary container in a Pod that supports the main container |
| **`pod.yaml`** | A YAML manifest declaring how a Pod should be configured and run |
| **`kubectl`** | The CLI for interacting with Kubernetes clusters |
| **`kubectl apply`** | Creates or updates a resource from a YAML file |
| **`kubectl get pods`** | Lists all pods in the current namespace |
| **`kubectl get pods -o wide`** | Extended pod list — shows node assignment and Pod IP |
| **`kubectl describe pod`** | Full status report for a pod — includes Events section |
| **`kubectl logs`** | Shows stdout/stderr output from a container |
| **`kubectl exec -it`** | Opens an interactive shell inside a running container |
| **`--previous` flag** | Shows logs from the previous container instance (useful after crashes) |
| **Cluster IP** | The internal IP assigned to a Pod — not reachable outside the cluster by default |
| **emptyDir** | A temporary volume that lives for the lifetime of the Pod — shared between containers |
| **Declarative** | Describe the desired state in a file — let Kubernetes figure out how to achieve it |
| **Imperative** | Tell the system exactly what to do step by step (e.g. `docker run` commands) |
| **KOPS State Store** | An S3 bucket where KOPS stores all cluster configuration |

---

## 📂 Summary of Tasks
- ✅ Understood: Dev tools (Minikube, k3s) vs production distributions — what's missing in dev tools.
- ✅ Learned: The major K8s distributions — Vanilla, OpenShift, Rancher, EKS, AKS, GKE.
- ✅ Understood: Cost strategy — self-managed for dev, managed services for production.
- ✅ Installed: KOPS prerequisites — Python 3, AWS CLI, kubectl.
- ✅ Configured: AWS credentials and S3 state store for KOPS.
- ✅ Understood: `kops create cluster` workflow and domain options (`.k8s.local` vs Route53).
- ✅ Understood: What a Pod is — wrapper around containers, declarative, version-controlled.
- ✅ Understood: Single vs multi-container Pods — shared network, shared storage, single IP.
- ✅ Set up: Local cluster with Minikube and verified with `kubectl get nodes`.
- ✅ Written: A `pod.yaml` manifest — `apiVersion`, `kind`, `metadata`, `spec.containers`.
- ✅ Deployed: First Pod with `kubectl apply -f pod.yaml`.
- ✅ Debugged: Using `kubectl describe pod` (Events section) and `kubectl logs`.
- ✅ Understood: Why Pods alone are not production-ready — no healing, scaling, or rolling updates.

---

## 💡 My Takeaway

The declarative vs imperative shift is the biggest mental change in moving from Docker to Kubernetes. With Docker, you type `docker run` with a long string of flags — it's imperative, procedural, and undocumented. With Kubernetes, you write a YAML file that says "here's what I want" — it's declarative, version-controlled, and reviewable. The YAML file is the documentation.

The multi-container Pod design also clicked — it's not about putting everything in one Pod. It's about co-locating containers that **must** be on the same node and **must** share storage or network. The sidecar pattern (app + log collector sharing a volume) is a perfect example. The alternative would be collecting logs over the network, which is more complex and fragile.

The KOPS S3 state store model is a direct parallel to Terraform's remote backend — externalizing state so that cluster management isn't tied to one engineer's laptop. Same problem, same solution pattern, different tool.

---

## 🔗 Resources
* [KOPS Documentation](https://kops.sigs.k8s.io/)
* [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
* [Kubernetes Pod Docs](https://kubernetes.io/docs/concepts/workloads/pods/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [EKS vs GKE vs AKS Comparison](https://www.stackrox.io/blog/eks-vs-gke-vs-aks/)
* [The Sidecar Pattern](https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
