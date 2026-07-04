![Progress](https://img.shields.io/badge/Progress-26%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 26 — Kubernetes ConfigMaps & Secrets

## 📝 Topic: Decoupling Configuration and Sensitive Data from Application Images
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 03, 2026

---

## 🎯 Learning Objectives
* Understand why configuration should never be hard-coded into an application image.
* Distinguish ConfigMaps (non-sensitive) from Secrets (sensitive) and know when to use each.
* Understand why Secrets are encrypted at rest in etcd while ConfigMaps are not.
* Implement ConfigMap data as environment variables using `configMapKeyRef`.
* Understand the core limitation of environment variables: no live updates without a restart.
* Implement ConfigMap data as a mounted volume for dynamic, restart-free updates.
* Create a generic Secret via `kubectl` and understand why base64 encoding is NOT encryption.
* Know when to reach for production-grade secret management (Vault, Sealed Secrets).

---

## 🗂️ Part 1 — ConfigMaps: Non-Sensitive Configuration

### The Problem They Solve

```
Without ConfigMaps:
  Port numbers, connection types, feature flags, etc.
  get hard-coded directly into application code or the image

  → Changing a config value means rebuilding the image
  → Same image can't be reused across dev/staging/prod
```

```
With ConfigMaps:
  Configuration lives OUTSIDE the image, injected at runtime
  → Same image works everywhere
  → Config changes don't require a rebuild
```

### Creating a ConfigMap

```yaml
# cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: django-config
data:
  DB_PORT: "5432"
  CONNECTION_TYPE: "postgres"
```

```bash
kubectl apply -f cm.yaml
kubectl get configmap django-config -o yaml
```

---

## 🔒 Part 2 — Secrets: Sensitive Data

### Why Secrets Exist Separately from ConfigMaps

```
Sensitive data (DB usernames, passwords, API keys) needs
different handling than a plain port number:

  ConfigMap → stored as plain text in etcd
  Secret    → encrypted at rest in etcd (when encryption is configured)
```

> **Best practice emphasized in the session:** Always pair Secrets with strict RBAC — following the **principle of least privilege** — so only the specific Service Accounts and users that genuinely need a Secret can read it. Encryption at rest doesn't help if every pod in the namespace has blanket read access.

---

## 🛠️ Part 3 — Practical Implementation: ConfigMap as Environment Variable

### Referencing a ConfigMap in a Deployment

```yaml
# deployment.yaml (Django app)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: django-app
  template:
    metadata:
      labels:
        app: django-app
    spec:
      containers:
        - name: django
          image: my-django-app:latest
          env:
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: django-config
                  key: DB_PORT
```

> **Correction noted during the session:** the correct field is `configMapKeyRef`, not a similarly-named field that's easy to mistype when writing this from memory — worth double-checking every time, since a typo here fails silently at apply time rather than throwing an obvious error.

### The Limitation: Environment Variables Are Immutable at Runtime

```
Sequence that exposes the problem:

1. Pod starts → env var DB_PORT=5432 injected at container start
2. ConfigMap is updated → DB_PORT changed to 5433
3. Pod is STILL running with DB_PORT=5432 in its environment

  → Environment variables are snapshotted at container start
  → They do NOT refresh live when the underlying ConfigMap changes
  → Only a pod restart re-reads the updated value
```

> **Why this matters:** for configuration that changes occasionally (feature flags, tunable thresholds), forcing a full pod restart on every change is often unacceptable — especially at scale.

---

## 📁 Part 4 — Volume Mounts: The Fix for Dynamic Updates

### Mounting a ConfigMap as a File Instead

```yaml
# deployment.yaml (volume mount version)
spec:
  containers:
    - name: django
      image: my-django-app:latest
      volumeMounts:
        - name: config-volume
          mountPath: /opt
  volumes:
    - name: config-volume
      configMap:
        name: django-config
```

```
Result inside the container:
  /opt/DB_PORT           → file containing "5432"
  /opt/CONNECTION_TYPE    → file containing "postgres"
```

### Why This Solves the Live-Update Problem

```
1. Pod starts → files mounted at /opt reflect current ConfigMap data
2. ConfigMap is updated → DB_PORT changed to 5433
3. Kubelet automatically refreshes the mounted files inside the pod
4. Application reads the new value from /opt/DB_PORT
   (if the app re-reads the file rather than caching it in memory once)

  → NO pod restart required
```

> **Key benefit demonstrated live:** volume-mounted ConfigMaps stay in sync automatically, making them the correct choice whenever configuration needs to change without disrupting running pods — environment variables remain fine for values that genuinely never change during a pod's lifetime.

---

## 🔑 Part 5 — Secrets in Action

### Creating a Generic Secret

```bash
kubectl create secret generic db-credentials \
  --from-literal=username=admin \
  --from-literal=password=SuperSecretPass123
```

```bash
kubectl get secret db-credentials -o yaml
# data:
#   username: YWRtaW4=
#   password: U3VwZXJTZWNyZXRQYXNzMTIz
```

### The Base64 Trap

```
kubectl automatically encodes Secret values in base64 by default.

CRITICAL misconception to avoid:
  base64 encoding ≠ encryption

  base64 is trivially reversible:
  echo "YWRtaW4=" | base64 --decode
  # admin

  Anyone with read access to the Secret object can decode it instantly.
```

> **Production recommendation from the session:** base64-encoded Secrets alone are not sufficient for real security. Tools like **HashiCorp Vault** (centralized secret management with dynamic secrets, leasing, and audit logging) or **Sealed Secrets** (encrypts secrets so they can be safely committed to Git and only decrypted inside the target cluster) should be used in production instead of relying on kubectl-created generic Secrets as the final security layer.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **ConfigMap** | A Kubernetes object storing non-sensitive configuration data, injectable as env vars or files |
| **Secret** | A Kubernetes object storing sensitive data, encrypted at rest in etcd (when configured) |
| **`configMapKeyRef`** | The field used to reference a specific ConfigMap key as an environment variable |
| **Environment Variable Injection** | Injecting config at container start; values are snapshotted and do NOT update live |
| **Volume Mount (ConfigMap)** | Mounting ConfigMap data as files inside a container; auto-refreshes on ConfigMap updates |
| **`kubectl create secret generic`** | CLI command to create a Secret from literal values or files |
| **Base64 Encoding** | A reversible encoding scheme — NOT encryption; Secrets default to this, which is a common misconception |
| **Principle of Least Privilege** | Granting only the minimum access necessary — core justification for pairing Secrets with strict RBAC |
| **HashiCorp Vault** | A production-grade secret management tool with dynamic secrets, leasing, and auditing |
| **Sealed Secrets** | A tool that encrypts Secrets so they can be safely stored in Git and decrypted only inside the target cluster |

---

## 📂 Summary of Tasks
- ✅ Understood: Why hard-coding configuration into images breaks portability across environments.
- ✅ Distinguished: ConfigMaps for non-sensitive data vs. Secrets for sensitive data.
- ✅ Understood: Secrets are encrypted at rest in etcd; ConfigMaps are not.
- ✅ Implemented: A ConfigMap injected into a Django pod as an environment variable via `configMapKeyRef`.
- ✅ Identified: The environment variable limitation — no live refresh without a pod restart.
- ✅ Implemented: The same ConfigMap mounted as files at `/opt`, confirming automatic refresh on updates.
- ✅ Created: A generic Secret via `kubectl create secret generic`.
- ✅ Understood: Base64 encoding is trivially reversible and is NOT a substitute for real encryption.
- ✅ Noted: HashiCorp Vault and Sealed Secrets as the recommended production-grade alternatives.
- ✅ Reinforced: RBAC + principle of least privilege as a mandatory companion to any Secret usage.

---

## 💡 My Takeaway

The environment-variable-vs-volume-mount comparison was the most immediately useful part of this session — it's the kind of distinction that looks like a minor implementation detail until a config change silently doesn't take effect in production and the "why isn't this updating" debugging starts. Knowing upfront that env vars are snapshotted at container start, while volume-mounted ConfigMaps refresh live via the kubelet, turns that into an architectural decision made at design time instead of a surprise later.

The base64 misconception is one I want to actively guard against — `kubectl create secret generic` *feels* secure because the output looks encoded and unreadable at a glance, but `base64 --decode` undoes that in one command. Treating that default behavior as "good enough" security would be a real gap. The instructor's framing — base64 protects against accidental glance-readability, not against anyone who actually wants the value — is the right mental model going forward, with Vault or Sealed Secrets as the actual production answer.

---

## 📈 Next Up
**Day 30:** Monitoring a Kubernetes cluster with Prometheus and Grafana — architecture, Helm-based setup, and Kube State Metrics for deployment/pod-level observability.

---

## 🔗 Resources
* [Kubernetes ConfigMaps Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)
* [Kubernetes Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
* [HashiCorp Vault](https://www.vaultproject.io/)
* [Sealed Secrets (Bitnami)](https://github.com/bitnami-labs/sealed-secrets)
* [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*