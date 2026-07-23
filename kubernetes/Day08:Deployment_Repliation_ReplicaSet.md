# Kubernetes — Day 08

## 🧩 The Big Picture

```
Deployment
   │
   └── manages → ReplicaSet
                     │
                     └── manages → Pods
```

Users typically create **Deployments**. Deployments create and manage **ReplicaSets**. ReplicaSets create and manage **Pods**. Each layer adds capability on top of the one below it.

---

## 1️⃣ Replication Controller (Legacy)

- **Purpose:** Ensures a specified number of pod replicas are running at all times — this is Kubernetes' **auto-healing** mechanism. If a pod crashes or is deleted, the controller spins up a replacement.
- **Load Balancing:** Acts as an endpoint for traffic, distributing requests across healthy pods.
- **Scalability:** Supports manual scaling by updating the `replicas` field, and can spread pods across multiple nodes to balance resource usage.
- **Selector limitation:** Only supports **equality-based** label selectors.
- **Status:** Considered a **legacy component** — deprecated in favor of ReplicaSet.

---

## 2️⃣ ReplicaSet (Modern Successor)

- **Purpose:** The modern replacement for the Replication Controller.
- **Key advantage:** Uses **selectors** (`matchLabels`), including **set-based** selectors, allowing it to manage pods that weren't necessarily created by the same ReplicaSet — more flexible than equality-based matching alone.

### Management Commands

```bash
# Imperative scaling
kubectl scale --replicas=<N> rs/<name>

# Live editing
kubectl edit rs <name>
```

---

## 3️⃣ Deployments (Higher-Level Abstraction)

- **Purpose:** Manages ReplicaSets and Pods, providing **declarative updates** to applications.
- **Rolling Updates:** The core benefit — update application versions (e.g., new container image) **without downtime** by gradually replacing pods one at a time.
- **Rollbacks:** Failed updates can be reverted easily.
- **History:** Deployments track revision history for auditing and rollback purposes.

### Key Commands

```bash
# Roll back to a previous revision
kubectl rollout undo deployment/<name>

# View revision history
kubectl rollout history deployment/<name>
```

---

## 🆚 Comparison Table

| Feature | Replication Controller | ReplicaSet | Deployment |
|---|---|---|---|
| Auto-healing | ✅ | ✅ | ✅ (via ReplicaSet) |
| Selector type | Equality-based only | Equality + set-based | Inherits from ReplicaSet |
| Rolling updates | ❌ | ❌ | ✅ |
| Rollbacks | ❌ | ❌ | ✅ |
| Revision history | ❌ | ❌ | ✅ |
| Status | Legacy/deprecated | Active, but rarely used directly | Standard practice |

> **Rule of thumb:** In real-world usage and on the CKA exam, you almost always work with **Deployments** — ReplicaSets and Replication Controllers exist underneath, but you rarely manage them directly.

---

## 💡 Hands-On Tips for CKA

- **Efficiency:** Use imperative commands with `--dry-run=client -o yaml` (e.g., via `kubectl create`) to generate YAML manifests quickly, then tweak as needed — much faster than writing from scratch under exam time pressure.
- **Cheat Sheet:** You don't need to memorize every command syntax — you need to know how to look it up efficiently using the official docs/cheat sheet during the exam.

---

## 🗂️ Glossary

| Term | Definition |
|---|---|
| **Replication Controller** | Legacy controller ensuring N pod replicas run at all times; equality-based selectors only |
| **ReplicaSet** | Modern successor to Replication Controller; supports set-based selectors |
| **Deployment** | Higher-level resource managing ReplicaSets; adds rolling updates, rollbacks, revision history |
| **Rolling Update** | Gradual pod replacement strategy enabling zero-downtime updates |
| **Rollback** | Reverting a Deployment to a previous revision |
| **matchLabels** | Selector mechanism used by ReplicaSets/Deployments to identify which pods they manage |

---

## 🔗 Cross-References
- Builds on Day 07 (Pods, imperative vs. declarative, YAML fundamentals)
- Sets up for upcoming topic: Services (exposing these managed pods to traffic)
- Complements DevOps/AWS track: rolling updates/rollbacks parallel blue-green and canary deployment strategies in CI/CD pipelines