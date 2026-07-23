# Kubernetes — Day 09

## 🎯 Why Services Exist

Pods are **ephemeral** — when a pod restarts, crashes, or gets rescheduled, it gets a **new IP address**. If other components communicated directly with pod IPs, every restart would break connectivity.

**Services** solve this by providing a **stable, persistent** endpoint that automatically tracks the current set of healthy pods behind it.

```
Client → Service (stable IP/DNS) → Endpoints → Pod IPs (changing)
```

---

## 🧩 Core Concepts & Terminology

| Term | Definition |
|---|---|
| **Target Port** | The port the application *inside the container* is actually listening on |
| **Port** | The internal port of the Service, used by other components within the cluster to reach the application |
| **Node Port** | A port on the underlying worker node (range: **30000–32767**) that allows external traffic from outside the cluster to reach the Service |
| **Endpoints** | The list of current, active pod IPs that the Service is load-balancing traffic to |

```
                 ┌─────────────────────────────┐
 External  ───►  │  NodePort (30000-32767)     │
 Traffic         │        ↓                     │
                 │      Service (Port)           │
                 │        ↓                     │
                 │   Endpoints (live pod IPs)    │
                 │        ↓                     │
                 │  Pod → Target Port (app)      │
                 └─────────────────────────────┘
```

---

## 🔌 Types of Kubernetes Services

### 1. ClusterIP (Default)
- Provides a stable **internal** IP for communication between pods within the cluster.
- Typical use: front-end pod talking to a back-end pod.
- Not accessible from outside the cluster.

### 2. NodePort
- Exposes the Service on **each node's IP** at a static port (30000–32767).
- Ideal for development or specific external access scenarios.
- Builds on top of ClusterIP (still gets an internal IP too).

### 3. LoadBalancer
- Provisions an **external load balancer**, typically via a cloud provider (AWS, Azure, GCP).
- Distributes traffic globally to your Service.
- Builds on top of NodePort/ClusterIP.

### 4. ExternalName
- Maps a Service to a **DNS name** instead of a selector.
- Useful for accessing external resources (e.g., an external database) via a consistent internal service name — no proxying, just DNS-level redirection.

### Comparison Table

| Type | Scope | Common Use Case |
|---|---|---|
| **ClusterIP** | Internal only | Pod-to-pod communication (default) |
| **NodePort** | Internal + external via node IP | Dev/test external access |
| **LoadBalancer** | Internal + external via cloud LB | Production external traffic |
| **ExternalName** | DNS-level mapping | Referencing external services by internal name |

---

## ⚠️ Key Implementation Notes

- **Case Sensitivity:** Kubernetes manifests are case-sensitive — service `type` values must be written exactly as expected (`NodePort`, `ClusterIP`, `LoadBalancer`, `ExternalName`).
- **Imperative Shortcut:** You can create a Service without writing a full YAML file:

```bash
kubectl expose deployment <deployment-name> --port=<port> --target-port=<target-port>
```

- **Kind Clusters:** When using Kind (Kubernetes in Docker), you must **manually configure extra port mappings** in your cluster config file to expose Services to your local machine — Kind doesn't automatically bridge NodePort access the way a cloud provider would.

```yaml
# kind-config.yaml (excerpt)
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 30080
```

---

## 💡 Productivity Tips for CKA

- Set up command aliases for speed, e.g.:

```bash
alias k=kubectl
```

- Enable **bash completion** for `kubectl` — highly recommended given the time pressure of the CKA exam.

---

## 🗂️ Glossary

| Term | Definition |
|---|---|
| **Service** | A stable network endpoint that routes traffic to a dynamic set of pods |
| **Target Port** | Port the app inside the container listens on |
| **Port** | Internal port exposed by the Service |
| **Node Port** | Node-level port (30000–32767) for external access |
| **Endpoints** | Live list of pod IPs currently backing a Service |
| **ClusterIP** | Default Service type; internal-only stable IP |
| **NodePort** | Exposes Service on every node's IP at a static port |
| **LoadBalancer** | Provisions a cloud load balancer for external traffic |
| **ExternalName** | Maps a Service to an external DNS name |

---

## 🔗 Cross-References
- Builds on Day 08 (Replication Controllers, ReplicaSets, Deployments) — Services expose the pods those controllers manage
- Builds on Day 06 (Kind cluster setup) — port mapping config ties directly into Kind's cluster YAML
- Sets up for upcoming topics: Ingress, Network Policies
- Complements DevOps/AWS track: Service types (NodePort/LoadBalancer) parallel ALB/NLB concepts in AWS networking