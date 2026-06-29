
![Progress](https://img.shields.io/badge/Progress-21%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 21 — Kubernetes Deployments & Services

## 📝 Topic: Auto-Healing with Deployments + Exposing Apps with Services

**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 27, 2026

---

## 🎯 Learning Objectives
* Understand the Container → Pod → Deployment hierarchy and why each layer exists.
* Explain how a Deployment creates and manages a ReplicaSet.
* Understand the desired state vs actual state control loop in practice.
* See auto-healing in action — delete a Pod, watch it come back.
* Scale a Deployment with zero downtime by editing the YAML.
* Explain why Pods alone can't reliably serve traffic (dynamic IPs, no load balancing).
* Know all three Service types — ClusterIP, NodePort, LoadBalancer — and when to use each.
* Understand how labels and selectors connect Services to Pods.

---

## 🏗️ Part 1 — The Three-Layer Hierarchy

```
Container
   ↓  (wrapped by)
  Pod
   ↓  (managed by)
Deployment
   ↓  (creates and owns)
ReplicaSet  →  manages → Pods
```

| Layer | What it is | Production use |
|---|---|---|
| **Container** | Basic execution unit — a Docker container | Never manage directly in K8s |
| **Pod** | Wrapper around containers — shared network/storage | Understand it, don't create it directly |
| **Deployment** | Manages Pods via ReplicaSet — provides healing and scaling | ✅ Always use this |

> **Rule:** You need to understand Pods to debug Kubernetes. You should almost never create a Pod directly. Always use a Deployment.

---

## 🔁 Part 2 — Deployments & ReplicaSets

### What Happens When You Create a Deployment

```
kubectl apply -f deployment.yaml
        ↓
Kubernetes creates a Deployment object
        ↓
Deployment automatically creates a ReplicaSet
        ↓
ReplicaSet creates the specified number of Pods
        ↓
Pods are scheduled onto worker nodes by the Scheduler
```

### The ReplicaSet Control Loop

The ReplicaSet is a **Kubernetes Controller** — it continuously watches the cluster and ensures:

```
Desired State (from deployment.yaml):   replicas: 3
Actual State  (what's running):         3 pods running

If actual ≠ desired:
  Actual = 2 (one pod crashed) → ReplicaSet creates 1 new pod
  Actual = 4 (extra pod found)  → ReplicaSet deletes 1 pod
```

This loop runs **continuously**, not just at deployment time. It's what makes auto-healing work.

---

## 📝 Part 3 — Writing a Deployment YAML

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # desired number of pods
  selector:
    matchLabels:
      app: nginx                 # ReplicaSet watches pods with this label
  template:                      # pod template — every pod created looks like this
    metadata:
      labels:
        app: nginx               # must match selector.matchLabels above
    spec:
      containers:
        - name: nginx
          image: nginx:1.24
          ports:
            - containerPort: 80
```

**The two critical label connections:**

```
spec.selector.matchLabels:  app: nginx   ← ReplicaSet uses this to find its pods
spec.template.metadata.labels: app: nginx ← pods are tagged with this

They MUST match. If they don't → ReplicaSet can't find its own pods.
```

---

## ⚡ Part 4 — Deploying and Verifying

### Deploy

```bash
kubectl apply -f deployment.yaml
```

```
deployment.apps/nginx-deployment created
```

### Verify All Resources Created

```bash
kubectl get all
```

```
NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-7d7f4b5c9-2xkpq   1/1     Running   0          30s
pod/nginx-deployment-7d7f4b5c9-8lmnt   1/1     Running   0          30s
pod/nginx-deployment-7d7f4b5c9-k9vwz   1/1     Running   0          30s

NAME                               READY   UP-TO-DATE   AVAILABLE
deployment.apps/nginx-deployment   3/3     3            3

NAME                                          DESIRED   CURRENT   READY
replicaset.apps/nginx-deployment-7d7f4b5c9   3         3         3
```

### Check the ReplicaSet Specifically

```bash
kubectl get rs
```

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-7d7f4b5c9    3         3         3       2m
```

---

## 🩺 Part 5 — Auto-Healing in Action

### Step 1: Note the Current Pods

```bash
kubectl get pods
# pod/nginx-deployment-7d7f4b5c9-2xkpq   Running
# pod/nginx-deployment-7d7f4b5c9-8lmnt   Running
# pod/nginx-deployment-7d7f4b5c9-k9vwz   Running
```

### Step 2: Manually Delete a Pod

```bash
kubectl delete pod nginx-deployment-7d7f4b5c9-2xkpq
```

### Step 3: Watch What Happens

```bash
kubectl get pods
# pod/nginx-deployment-7d7f4b5c9-8lmnt   Running   ← original
# pod/nginx-deployment-7d7f4b5c9-k9vwz   Running   ← original
# pod/nginx-deployment-7d7f4b5c9-r4tpx   Running   ← NEW — auto-created by ReplicaSet
```

```
Timeline:
  T+0s:  Pod deleted
  T+1s:  ReplicaSet detects: "Actual=2, Desired=3 → gap of 1"
  T+2s:  ReplicaSet creates a new pod
  T+5s:  New pod is scheduled and starts
  T+10s: New pod is Running

Downtime: effectively zero (2 pods still served traffic throughout)
```

This is the **desired state vs actual state** control loop working in real time.

---

## 📈 Part 6 — Scaling with Zero Downtime

### Scale Up — Edit the YAML

```yaml
# deployment.yaml
spec:
  replicas: 5    # changed from 3 to 5
```

```bash
kubectl apply -f deployment.yaml
```

```bash
kubectl get pods
# 5 pods now running — 2 new ones added without removing the existing 3
# No downtime. No traffic interruption.
```

### Scale Down

```yaml
spec:
  replicas: 2    # changed from 5 to 2
```

```bash
kubectl apply -f deployment.yaml
# Kubernetes gracefully terminates 3 pods, keeping 2 running
```

### Scale via Command (without editing YAML)

```bash
kubectl scale deployment nginx-deployment --replicas=6
```

> ⚠️ **Important:** Scaling via command doesn't update the YAML file. If you then `kubectl apply -f deployment.yaml`, it will revert to the YAML value. Always keep the YAML as the source of truth and commit it to Git.

---

## 🌐 Part 7 — Why Pods Need Services

### Problem 1 — Dynamic IP Addresses

```
Before auto-healing:
  Pod A: IP 10.244.0.5  ← client hardcodes this IP
  
Pod A crashes, ReplicaSet creates Pod B:
  Pod B: IP 10.244.0.8  ← completely new IP

Client → 10.244.0.5 → connection refused (pod gone)
Client doesn't know about 10.244.0.8

Without a Service: the client has no reliable way to reach the application
```

### Problem 2 — No Load Balancing

```
Deployment has 3 replicas:
  Pod A: 10.244.0.5
  Pod B: 10.244.0.6
  Pod C: 10.244.0.7

Without a Service:
  Client must pick one IP manually
  No distribution across all three
  One pod gets all traffic, others idle
```

### Problem 3 — No External Access

```
Pod IPs are Cluster IPs — only reachable inside the cluster
Without a Service, no external traffic can reach the application
```

---

## 🔧 Part 8 — How Kubernetes Services Work

### Labels and Selectors — The Core Mechanism

Services don't track Pods by IP. They track Pods by **labels**.

```
Pods (created by Deployment):
  Pod A: labels: { app: payment }  IP: 10.244.0.5
  Pod B: labels: { app: payment }  IP: 10.244.0.6
  Pod C: labels: { app: payment }  IP: 10.244.0.7

Service:
  selector: { app: payment }  ← "send traffic to all pods with this label"

Pod A crashes → Pod D created with IP 10.244.0.9 and label app: payment
Service automatically routes traffic to Pod D
No reconfiguration. No manual update.
```

The Service never tracks IPs. It tracks labels. Labels don't change when Pods are recreated.

---

## 📋 Part 9 — Writing a Service YAML

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx           # matches pods with this label
  ports:
    - protocol: TCP
      port: 80           # port on the Service (what clients connect to)
      targetPort: 80     # port on the Pod (where the app listens)
  type: ClusterIP        # default — internal only
```

```bash
kubectl apply -f service.yaml

kubectl get svc
# NAME            TYPE        CLUSTER-IP      PORT(S)   AGE
# nginx-service   ClusterIP   10.96.45.123    80/TCP    30s
```

Now instead of `10.244.0.5` (Pod IP that changes), clients use `10.96.45.123` (Service IP that's stable).

---

## 🌍 Part 10 — The Three Service Types

### ClusterIP (Default)

```
Accessible: inside the cluster only

                  Cluster boundary
                  ┌──────────────────────────────┐
  External user ✗ │  Service (ClusterIP)         │
                  │       ↓                      │
                  │  Pod A | Pod B | Pod C       │
                  └──────────────────────────────┘

Use case:
  → Backend API accessed only by the frontend (same cluster)
  → Database accessed only by the application (same cluster)
  → Internal microservice-to-microservice communication
```

```yaml
spec:
  type: ClusterIP    # or omit — ClusterIP is the default
```

---

### NodePort

```
Accessible: from within the organization's network (not public internet)

External user → <Node IP>:<NodePort> → Service → Pod

                  Cluster boundary
                  ┌────────────────────────────────────┐
  Internal user → │  Node IP: 192.168.1.10             │
                  │  NodePort: 30080                   │
                  │       ↓                            │
                  │  Service → Pod A | Pod B | Pod C   │
                  └────────────────────────────────────┘

NodePort range: 30000–32767 (Kubernetes reserved range)

Use case:
  → Internal tools accessible to the engineering team
  → Staging environment accessible within the VPN
```

```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080     # optional — K8s assigns one if omitted
```

---

### LoadBalancer

```
Accessible: public internet

                              Cloud Provider (AWS/Azure/GCP)
Public internet user → ELB / ALB (Public IP) → Service → Pod A | Pod B | Pod C
                                ↑
                    Cloud Controller Manager provisions this automatically

Use case:
  → Customer-facing applications (e-commerce, SaaS)
  → Public APIs
  → Any service that needs to be reachable from the internet
```

```yaml
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl get svc
# NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)
# nginx-service   LoadBalancer   10.96.45.123    54.210.xxx.xxx    80:32145/TCP
#                                                ↑
#                                     AWS provisioned this public IP automatically
```

---

### Service Types Summary

| Type | Accessible From | When to Use |
|---|---|---|
| **ClusterIP** | Inside cluster only | Internal service-to-service communication |
| **NodePort** | Organization network (VPN/intranet) | Internal tools, staging, dev access |
| **LoadBalancer** | Public internet | Customer-facing production services |

```
Amazon.com example:
  Frontend:     LoadBalancer → public internet users
  Payment API:  ClusterIP    → only frontend can reach it
  Database:     ClusterIP    → only payment API can reach it
  Admin panel:  NodePort     → only engineers on VPN
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Deployment** | A K8s resource that manages Pods via a ReplicaSet — provides auto-healing and scaling |
| **ReplicaSet (RS)** | A Kubernetes controller that ensures the desired number of pod replicas are always running |
| **Desired State** | The configuration declared in YAML — what K8s should maintain |
| **Actual State** | What is currently running in the cluster |
| **Control Loop** | The continuous observe → compare → act cycle that reconciles desired and actual state |
| **Auto-Healing** | Automatic replacement of crashed pods to maintain desired replica count |
| **`kubectl get all`** | Lists all resources (pods, deployments, replicasets, services) in the current namespace |
| **`kubectl get rs`** | Lists all ReplicaSets |
| **`kubectl apply -f`** | Creates or updates resources from a YAML file |
| **`kubectl scale`** | Scales a deployment's replica count via command line |
| **Service** | A stable endpoint that routes traffic to a set of Pods, selected by labels |
| **Label** | A key-value pair attached to a K8s object — used for grouping and selection |
| **Selector** | A Service's filter — routes traffic to Pods matching the specified labels |
| **ClusterIP** | Default Service type — accessible only within the cluster |
| **NodePort** | Service type — accessible on a port on each worker node (30000–32767 range) |
| **LoadBalancer** | Service type — provisions a cloud load balancer with a public IP |
| **Cloud Controller Manager** | K8s component that requests cloud resources (e.g. AWS ELB) on behalf of Services |
| **`targetPort`** | The port on the Pod where the application actually listens |
| **`port`** | The port exposed by the Service — what clients connect to |
| **`nodePort`** | The port on each Node for NodePort Services — in the 30000-32767 range |

---

## 📂 Summary of Tasks
- ✅ Understood: Container → Pod → Deployment hierarchy and why each layer exists.
- ✅ Understood: Deployment creates a ReplicaSet which manages Pods.
- ✅ Written: `deployment.yaml` with `replicas`, `selector`, and `template` sections.
- ✅ Deployed: Nginx deployment and verified with `kubectl get all` and `kubectl get rs`.
- ✅ Observed: Auto-healing in action — deleted a pod, watched ReplicaSet recreate it.
- ✅ Scaled: Deployment from 3 to 5 replicas with zero downtime via YAML update.
- ✅ Understood: Why Pods need Services — dynamic IPs, no load balancing, no external access.
- ✅ Understood: Labels and selectors — how Services track Pods without hardcoding IPs.
- ✅ Written: `service.yaml` connecting to Pods via label selector.
- ✅ Understood: All three Service types — ClusterIP, NodePort, LoadBalancer — and when to use each.

---

## 💡 My Takeaway

The auto-healing demo made the desired state model concrete in a way that architecture diagrams don't. Delete a pod. Watch it come back in seconds. That's the loop working — not in theory, but actually. The ReplicaSet sees a gap between desired (3) and actual (2) and immediately closes it.

The label-based Service discovery is the other insight that changed how I think about Kubernetes networking. A Service doesn't care what IP a Pod has. It only cares that the Pod has the right label. When a Pod crashes and comes back with a new IP, the Service automatically routes to it — because the label is still `app: payment`. Stable identity through labels, not through IPs. That's a fundamentally different model from traditional networking.

The three Service types also map cleanly to the three visibility tiers every organization has: private (ClusterIP — internal only), organizational (NodePort — VPN/intranet), and public (LoadBalancer — internet). Choosing the wrong type is a real security issue — accidentally setting a database Service to LoadBalancer would expose it to the internet.

---

## 📈 Next Up
**Day 24:** Kubernetes ConfigMaps & Secrets — externalizing configuration and managing sensitive data without hardcoding into container images.

---

## 🔗 Resources
* [Kubernetes Deployments Docs](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Services Docs](https://kubernetes.io/docs/concepts/services-networking/service/)
* [ReplicaSet Docs](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
* [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
