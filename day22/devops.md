![Progress](https://img.shields.io/badge/Progress-33%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 22 — Kubernetes Mock Interview: 10 Core Questions

## 📝 Topic: Theoretical Concepts, Common Interview Questions & Real DevOps Day-to-Day
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 29, 2026

---

## 🎯 Learning Objectives
* Articulate Docker vs Kubernetes clearly enough for an interview setting.
* Recite the full Kubernetes architecture from memory — Control Plane and Data Plane.
* Explain Docker Swarm vs Kubernetes and why K8s won at enterprise scale.
* Define Containers vs Pods precisely.
* Explain Namespaces and why logical isolation matters in shared clusters.
* Describe Kube-proxy's networking role with iptables.
* Recall all three Service types and their access scope.
* Explain Kubelet's responsibility on worker nodes.
* Know what a real DevOps engineer's day-to-day actually looks like.

---

## 🎤 Q1 — Docker vs Kubernetes

**The interview-ready answer:**

> "Docker is a containerization platform — it builds, runs, and ships individual containers. Kubernetes is a container orchestration platform — it manages containers at scale across a cluster of machines. The key differentiators are auto-healing, auto-scaling, and built-in load balancing, none of which Docker provides natively."

```
Docker alone:
  ✅ Build images, run containers
  ❌ No multi-host distribution
  ❌ No auto-healing
  ❌ No auto-scaling
  ❌ No built-in load balancing

Kubernetes:
  ✅ Everything Docker does (uses a container runtime under the hood)
  ✅ Distributes containers across a cluster of nodes
  ✅ Auto-healing — ReplicaSet recreates crashed pods
  ✅ Auto-scaling — HPA adjusts replica count based on load
  ✅ Load balancing — Services distribute traffic automatically
```

---

## 🎤 Q2 — Kubernetes Architecture

**The interview-ready answer:**

> "Kubernetes architecture splits into two planes. The Control Plane is the brain — it contains the API Server, Scheduler, etcd, Controller Manager, and Cloud Controller Manager. The Data Plane is where workloads actually run — it consists of Kubelet, Kube-proxy, and the Container Runtime on every worker node."

```
CONTROL PLANE (decisions)          DATA PLANE (execution)
─────────────────────────          ──────────────────────
API Server      → gateway          Kubelet          → node agent
Scheduler        → placement        Kube-proxy       → networking
etcd              → state store     Container Runtime → runs containers
Controller Mgr    → reconciliation
Cloud Controller  → cloud API bridge
  Manager (CCM)
```

**One-line summary of each component:**

| Component | One-line role |
|---|---|
| API Server | Single entry point for all cluster operations |
| Scheduler | Decides which node a new pod runs on |
| etcd | Stores the entire cluster state |
| Controller Manager | Runs control loops to reconcile desired vs actual state |
| CCM | Bridges Kubernetes to cloud provider APIs (only on managed clusters) |
| Kubelet | Ensures pods on this node match their spec |
| Kube-proxy | Manages network rules and load balances traffic to pods |
| Container Runtime | Actually starts and stops containers |

---

## 🎤 Q3 — Docker Swarm vs Kubernetes

**The interview-ready answer:**

> "Both are container orchestration tools, but Kubernetes wins at enterprise scale. Docker Swarm is simpler to set up and has a gentler learning curve, but Kubernetes has a much larger community, broader ecosystem support through CNCF, and Custom Resource Definitions that let you extend the platform for enterprise-specific needs. Most large organizations standardize on Kubernetes because of that ecosystem maturity."

| Factor | Docker Swarm | Kubernetes |
|---|---|---|
| **Setup complexity** | Simple | More complex |
| **Learning curve** | Gentle | Steeper |
| **Community/ecosystem** | Smaller | Massive (CNCF) |
| **Extensibility** | Limited | CRDs, Operators, full API extension |
| **Enterprise adoption** | Declining | Industry standard |
| **Auto-scaling** | Basic | Sophisticated (HPA, VPA, Cluster Autoscaler) |

---

## 🎤 Q4 — Containers vs Pods

**The interview-ready answer:**

> "A container is the basic unit of execution — what Docker runs. A Pod is the smallest deployable unit in Kubernetes, and it's a runtime specification wrapper around one or more containers. Kubernetes never schedules a bare container — it always schedules a Pod, which provides shared networking and storage for the containers inside it."

```
Container:  process + filesystem + isolated namespace
            managed via Docker commands directly

Pod:        Kubernetes' wrapper around 1+ containers
            Provides:
              → Shared network namespace (single Cluster IP)
              → Shared storage volumes
              → Co-scheduling (always on the same node)
            Defined declaratively via pod.yaml
```

---

## 🎤 Q5 — Namespaces

**The interview-ready answer:**

> "Namespaces provide logical isolation of resources within a single Kubernetes cluster. They let multiple teams or projects share the same physical cluster without their resources colliding — you can have a `frontend` namespace and a `backend` namespace with identically-named resources, and Kubernetes keeps them completely separate."

```
One physical cluster:
  ┌─────────────────────────────────────────────────┐
  │  Namespace: dev-team-a                          │
  │    Deployment: api    Service: api-svc           │
  ├─────────────────────────────────────────────────┤
  │  Namespace: dev-team-b                          │
  │    Deployment: api    Service: api-svc           │  ← same names, no conflict
  ├─────────────────────────────────────────────────┤
  │  Namespace: production                          │
  │    Deployment: api    Service: api-svc           │
  └─────────────────────────────────────────────────┘
```

```bash
# Create a namespace
kubectl create namespace dev-team-a

# Deploy to a specific namespace
kubectl apply -f deployment.yaml -n dev-team-a

# List resources in a namespace
kubectl get pods -n dev-team-a

# Default namespaces in every cluster
kubectl get namespaces
# default            → where resources go if no namespace specified
# kube-system        → core Kubernetes components (API server, etc.)
# kube-public        → publicly readable resources
# kube-node-lease    → node heartbeat data
```

---

## 🎤 Q6 — Kube-Proxy

**The interview-ready answer:**

> "Kube-proxy runs on every worker node and maintains network rules that route traffic to the correct pods. It manages iptables entries so that when traffic hits a Service, it gets load-balanced across all the pods backing that service — regardless of which node those pods are actually running on."

```
Service "payment-api" → backed by 3 pods on 3 different nodes

Kube-proxy on every node maintains iptables rules:
  Request to Service IP → randomly forwarded to one of the 3 pod IPs
  → load balancing happens at the network layer, transparently

If a pod moves or is replaced:
  → Kube-proxy updates iptables rules automatically
  → No application-level changes needed
```

---

## 🎤 Q7 — Service Types

**The interview-ready answer:**

> "Kubernetes has three primary Service types. ClusterIP is the default and only exposes the application inside the cluster — used for internal service-to-service communication. NodePort exposes the app on a specific port on every node, accessible within the organization's network. LoadBalancer requests a public IP from the cloud provider, making the application accessible from the public internet."

| Type | Scope | Real-world example |
|---|---|---|
| **ClusterIP** | Internal cluster only | Database accessed only by the backend API |
| **NodePort** | Organization network | Staging environment accessible via VPN |
| **LoadBalancer** | Public internet | Customer-facing e-commerce frontend |

```bash
kubectl get svc
NAME            TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)
backend-svc     ClusterIP      10.96.45.1    <none>           8080/TCP
staging-svc     NodePort       10.96.45.2    <none>           80:30080/TCP
frontend-svc    LoadBalancer   10.96.45.3    54.210.xxx.xxx   80:32145/TCP
```

---

## 🎤 Q8 — Kubelet

**The interview-ready answer:**

> "Kubelet is the agent that runs on every worker node. It's responsible for managing the lifecycle of pods scheduled to that node — starting them, monitoring their health, and restarting them if they fail. It continuously reports node and pod status back to the API Server, which is how the control plane knows the actual state of the cluster."

```
Kubelet's core loop on every node:
  1. Watch the API Server for pods assigned to this node
  2. Tell the Container Runtime to start/stop containers as needed
  3. Continuously check: are pods running as specified?
  4. Report status back to the API Server
  5. If a pod's container crashes → restart it locally first
  6. If the pod itself is unhealthy beyond repair → report to control plane
```

> **Distinction worth knowing:** Kubelet handles container-level restarts on the same node. The ReplicaSet (Controller Manager) handles pod-level replacement — like when an entire node fails and pods need to be rescheduled elsewhere.

---

## 🎤 Q9 — A Realistic Day-to-Day for a DevOps Engineer

This is the question that separates candidates who've memorized concepts from those who understand the role.

```
A typical day might include:

Morning:
  → Check overnight alerts (CloudWatch, Datadog, PagerDuty)
  → Review any failed CI/CD pipeline runs
  → Standup — report blockers, sync with the team

Throughout the day:
  → Cluster management — node health, resource utilization
  → Monitoring and observability — dashboards, log analysis
  → Troubleshooting — a service is slow, a pod keeps crashing
  → Node maintenance — patching, scaling node pools
  → Supporting developers — answering "why won't my pod start?"
                           — reviewing Dockerfiles and K8s manifests
                           — acting as the infrastructure subject matter expert

Ongoing:
  → Writing/improving Terraform modules
  → Writing/improving CI/CD pipelines
  → Incident response when something breaks
  → Capacity planning — are we about to run out of resources?
```

> **Why this matters for interviews:** Candidates who describe DevOps purely as "deploying with Kubernetes" sound junior. Candidates who can talk about troubleshooting a crashing pod at 2am, or explaining to a developer why their deployment is stuck in `ImagePullBackOff`, sound like they've actually done the job.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Container Orchestration** | Automated management of containers at scale — scheduling, scaling, healing |
| **Control Plane** | The decision-making layer of Kubernetes — API Server, Scheduler, etcd, Controllers |
| **Data Plane** | The execution layer — worker nodes running Kubelet, Kube-proxy, Container Runtime |
| **Docker Swarm** | Docker's native orchestration tool — simpler but less widely adopted than K8s |
| **CRD** | Custom Resource Definition — extends the Kubernetes API for enterprise-specific needs |
| **Pod** | The smallest deployable unit — a runtime wrapper around one or more containers |
| **Namespace** | A logical partition within a cluster — isolates resources between teams/projects |
| **`kube-system`** | The default namespace containing core Kubernetes components |
| **Kube-proxy** | Manages iptables network rules — routes Service traffic to backing pods |
| **iptables** | The Linux kernel firewall/routing tool Kube-proxy configures for traffic management |
| **ClusterIP** | Default Service type — internal cluster access only |
| **NodePort** | Service type exposing a port on every node — organizational network access |
| **LoadBalancer** | Service type provisioning a cloud load balancer — public internet access |
| **Kubelet** | The node agent — manages pod lifecycle and reports status to the API Server |
| **`ImagePullBackOff`** | A common pod error — Kubernetes can't pull the specified container image |
| **Subject Matter Expert (SME)** | The role a DevOps engineer plays when supporting developers on infrastructure questions |

---

## 📂 Summary of Tasks
- ✅ Practiced: Docker vs Kubernetes — articulated in interview-ready language.
- ✅ Recited: Full K8s architecture — Control Plane and Data Plane components.
- ✅ Compared: Docker Swarm vs Kubernetes — why K8s wins at enterprise scale.
- ✅ Defined: Containers vs Pods precisely.
- ✅ Explained: Namespaces — logical isolation for multi-team clusters.
- ✅ Explained: Kube-proxy's role — iptables-based traffic routing.
- ✅ Recalled: All 3 Service types — ClusterIP, NodePort, LoadBalancer with real examples.
- ✅ Explained: Kubelet's node-level responsibilities and reporting loop.
- ✅ Understood: What real DevOps day-to-day work actually looks like beyond deployments.

---

## 💡 My Takeaway

Going through this as a mock interview rather than as a tutorial changed how I processed the material. Knowing what a Namespace is and being able to explain it in 15 seconds in a way that demonstrates real understanding are different skills. The interview-ready answers in this session forced me to compress each concept down to its essential point — what problem does this solve, and why does it exist.

The Kubelet vs ReplicaSet distinction was the most useful clarification today. Both deal with "keeping pods running," but at different layers: Kubelet handles container restarts on the same node (a process crashed, restart it locally). ReplicaSet handles pod-level replacement, often across nodes (a pod or even an entire node is gone, schedule a replacement elsewhere). Conflating these two in an interview would be a clear signal of surface-level understanding.

The day-to-day question is the one I'll remember longest. It's tempting to think of DevOps work as "writing YAML and deploying things." The honest answer — troubleshooting, supporting developers, node maintenance, incident response, capacity planning — is a much more accurate and much less glamorous picture. That's worth internalizing before walking into any real interview.

---

## 📈 Next Up
**Day 25:** Kubernetes ConfigMaps & Secrets in depth, plus a second round of mock interview questions covering storage, Ingress, and RBAC.

---

## 🔗 Resources
* [Kubernetes Official Docs — Concepts](https://kubernetes.io/docs/concepts/)
* [Kubernetes Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
* [CNCF — Kubernetes vs Docker Swarm](https://www.cncf.io/blog/)
* [Common Kubernetes Interview Questions — Curated List](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Troubleshooting Kubernetes Pods](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*