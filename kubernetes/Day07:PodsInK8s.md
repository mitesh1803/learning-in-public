# Kubernetes — Day 07

## 🧱 Introduction to Pods

- A **Pod** is the smallest deployable unit in Kubernetes.
- Typically represents a **single container**, though it can hold multiple tightly-coupled, collaborating containers running together on a worker node.
- All user interaction with the cluster happens through the **`kubectl`** client, which talks to the Kubernetes **API server**.

```
User → kubectl → API Server → Scheduler → Worker Node → Pod → Container(s)
```

---

## ⚖️ Imperative vs. Declarative Management

| Approach | How it works | Best for |
|---|---|---|
| **Imperative** | Run direct commands telling Kubernetes exactly what to do (e.g., `kubectl run ...`) | Quick troubleshooting, local testing, one-off tasks |
| **Declarative** | Define the **desired state** in a YAML/JSON file, apply it with `kubectl apply` or `kubectl create` | Production, CI/CD pipelines, repeatable/version-controlled infra |

> **Mental model:** Imperative = "do this now." Declarative = "make the cluster look like this, however it needs to get there."

---

## 📄 Working with YAML

### Syntax Rules
- YAML relies on **indentation and spacing** — double spaces are recommended.
- YAML is **case-sensitive**.
- Structure matters more than in JSON — misaligned indentation breaks the manifest.

### The Four Core Top-Level Fields

Every Kubernetes manifest requires these:

```yaml
apiVersion: v1        # Which K8s API version to use
kind: Pod              # The type of object being defined
metadata:               # Name, labels, namespace, etc.
  name: my-pod
  labels:
    app: my-app
spec:                    # The desired configuration
  containers:
    - name: my-container
      image: nginx
      ports:
        - containerPort: 80
```

| Field | Purpose |
|---|---|
| `apiVersion` | The version of the Kubernetes API to use (e.g., `v1`) |
| `kind` | The type of object (e.g., `Pod`, `Deployment`, `Service`) |
| `metadata` | Info like name, labels, and namespace |
| `spec` | The desired configuration (container images, ports, etc.) |

### Discovering Fields
Use `kubectl explain <object>` to pull up documentation for any object's fields directly from the CLI — no need to search docs for basic field lookups.

```bash
kubectl explain pod
kubectl explain pod.spec.containers
```

---

## 🔧 Key `kubectl` Commands

### Creation

```bash
# Imperative
kubectl run <name> --image=<image>

# Declarative
kubectl apply -f <file.yaml>
```

### Troubleshooting

```bash
# View status, events, and error logs
kubectl describe pod <name>

# Directly edit a running pod's configuration
kubectl edit pod <name>
```

### Management

```bash
# List all pods
kubectl get pods

# Extended info — includes pod IP and node name
kubectl get pods -o wide

# Open an interactive shell inside the container
kubectl exec -it <pod_name> -- <command>

# Remove a pod
kubectl delete pod <name>
```

---

## 💡 Pro-Tip: Generating YAML Without Writing It From Scratch

Instead of hand-writing complex manifests, combine an imperative command with `--dry-run=client -o yaml` to generate a starter template:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

This creates a valid, ready-to-edit YAML file without actually creating the pod — a huge time-saver for both daily work and the CKA exam.

---

## 🗂️ Glossary

| Term | Definition |
|---|---|
| **Pod** | Smallest deployable unit in Kubernetes; wraps one or more containers |
| **Imperative** | Direct command-based cluster management |
| **Declarative** | Config-file-based management describing desired state |
| **Manifest** | A YAML/JSON file describing a Kubernetes object |
| **apiVersion** | Specifies which version of the K8s API to use for an object |
| **dry-run** | Simulates a command without actually executing it against the cluster |

---

## 🔗 Cross-References
- Builds on Day 06 (Kind cluster setup, kubectl, context switching)
- Sets up for upcoming topics: Deployments, Services, and multi-container Pod patterns
- Complements DevOps/AWS track: YAML manifests parallel Terraform's declarative infra-as-code approach