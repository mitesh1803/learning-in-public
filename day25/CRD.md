![Progress](https://img.shields.io/badge/Progress-25%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 28 — Kubernetes Custom Resources, CRDs & Custom Controllers

## 📝 Topic: Extending the Kubernetes API Beyond Native Resources
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 02, 2026

---

## 🎯 Learning Objectives
* Understand why native Kubernetes resources (Pods, Deployments, Services) aren't enough for many real-world tools.
* Learn the three core components of Kubernetes extensibility: CRD, Custom Resource, and Custom Controller.
* Understand how a Custom Controller actually watches and reacts to changes in the cluster.
* Understand why Golang and `client-go` are the standard choice for writing controllers.
* Learn the role of the Kubernetes Controller Runtime, Watchers, and worker queues.
* Understand how CRDs and Controllers are typically deployed in production (Helm).
* Know where to look for real-world examples: CNCF projects and the Kubernetes Sample Controller repo.

---

## ❓ Part 1 — Why Extend Kubernetes?

### The Gap Native Resources Leave

```
Kubernetes ships with:
  Pods, Deployments, Services, ConfigMaps, Ingress, etc.

These cover generic workload orchestration —
but NOT specialized, domain-specific logic.
```

```
Real-world tools that need MORE than native resources offer:

  Istio      → Service Mesh (traffic shaping, mTLS, retries, circuit breaking)
  Argo CD    → GitOps (continuously reconciling cluster state from a Git repo)
  Keycloak   → Identity & Access Management (realms, clients, users as k8s objects)
```

> **The core problem:** None of this logic exists in the native Kubernetes control plane. `kubectl apply` on a Deployment doesn't know how to configure mTLS between services or sync from a Git repo — that behavior has to be taught to the cluster somehow.

```
The answer: Kubernetes' API is EXTENSIBLE.
Instead of forking Kubernetes itself, you can register
entirely new resource types and teach the cluster new behavior
without touching a single line of core Kubernetes code.
```

---

## 🧩 Part 2 — The Three Core Components

### 1. Custom Resource Definition (CRD) — The Schema

```
A CRD is a YAML definition that tells the Kubernetes API Server:
  "A new type of object exists now. Here's its name,
   its fields, and how to validate it."
```

```yaml
# Example: CRD for a fictional "IstioRoute" custom resource
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: istioroutes.networking.example.com
spec:
  group: networking.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                host:
                  type: string
                weight:
                  type: integer
  scope: Namespaced
  names:
    plural: istioroutes
    singular: istioroute
    kind: IstioRoute
```

> Once this is applied, `kubectl get istioroutes` becomes a valid command — Kubernetes now understands this as a real API type, exactly like it understands `pods` or `deployments` natively.

### 2. Custom Resource (CR) — The Instance

```
The CRD is the schema.
The Custom Resource is an actual OBJECT created against that schema.
```

```yaml
# Example: An actual IstioRoute instance (a Custom Resource)
apiVersion: networking.example.com/v1
kind: IstioRoute
metadata:
  name: payments-route
spec:
  host: payments.internal
  weight: 80
```

```bash
kubectl apply -f istioroute-instance.yaml
kubectl get istioroutes
# NAME              AGE
# payments-route    5s
```

> At this point, Kubernetes has validated and stored this object — but nothing has actually *happened* yet. The API Server just knows the desired state exists.

### 3. Custom Controller — The Logic That Acts

```
The CRD + CR only describe INTENT.
Something has to actually DO something about it.
That "something" is the Custom Controller.
```

```
Custom Controller responsibilities:
  → Continuously WATCH the cluster for Custom Resources
    of a given type (Create / Update / Delete events)
  → When a change is detected, run the actual logic
    (e.g., configure Istio's Envoy proxies to match the new route)
  → Keep reconciling until observed state == desired state
```

> **The key mental model:** CRD = new API type registered. CR = a specific desired-state object. Controller = the active loop that makes reality match that desired state. Without the controller, a CR is just inert data sitting in etcd.

---

## 🛠️ Part 3 — How to Write a Custom Controller

### Why Golang

```
Reasons Go is the default choice:
  → Kubernetes itself is written entirely in Go
  → client-go is Kubernetes' official Go client library,
    offering deep, well-supported access to the API Server
  → Best documentation, best community examples, best long-term support
```

### You Don't Build the Watch Loop From Scratch

```
Kubernetes Controller Runtime provides pre-built primitives:

  Watcher    → subscribes to API Server events for a resource type
  Informer   → caches objects locally, reduces API Server load
  Listener   → reacts when a Watcher detects a change
```

### The Reconciliation Workflow

```
1. A Custom Resource is Created / Updated / Deleted
        ↓
2. The Watcher detects this change via the API Server
        ↓
3. The affected object's key is placed into a WORKER QUEUE
        ↓
4. A worker goroutine pulls the item off the queue
        ↓
5. The controller's "Reconcile" logic runs:
     - Reads current state
     - Compares against desired state (the CR's spec)
     - Takes whatever action closes the gap
        ↓
6. Loop continues indefinitely, watching for the next change
```

> **Why a queue instead of acting immediately inline:** it decouples "detecting a change" from "processing a change," allows retries on failure, and prevents the controller from being overwhelmed by rapid bursts of events on the same object.

---

## 🚀 Part 4 — Practical Implementation

### Deployment: CRDs + Controllers Together via Helm

```
Most production setups package BOTH pieces into a single Helm chart:
  → CRD YAML(s) — registers the new API type
  → Controller Deployment — the actual running application/logic

helm install my-operator ./my-operator-chart
```

```
Why Helm specifically:
  → CRDs must exist in the cluster BEFORE any CR of that type
    can be created — ordering matters
  → Helm handles this install ordering and versioning cleanly
  → Upgrades to the CRD schema and controller logic can be
    version-locked together
```

### Debugging Custom Controllers (The DevOps Day-to-Day)

```bash
# Check what the controller itself is doing/logging
kubectl logs -n <controller-namespace> <controller-pod-name>

# Inspect a specific Custom Resource's current state and events
kubectl describe istioroute payments-route

# Check overall CR status across the cluster
kubectl get istioroutes -A
```

> **Reality check from the session:** as a DevOps engineer, you're far more likely to be *debugging* a misbehaving controller (stuck reconciliation, stale status, crash-looping controller pod) than writing one from scratch — understanding the CR → Controller → Reconcile flow is what makes that debugging possible instead of guesswork.

### Where to Learn From Real Examples

```
Recommended resources from the session:
  → CNCF (Cloud Native Computing Foundation) website
    - Browse industry-standard operators/controllers:
      Argo CD, Istio, Prometheus Operator, etc.
  → Kubernetes Sample Controller repository
    - github.com/kubernetes/sample-controller
    - A minimal, official reference implementation showing
      the full CRD + Controller pattern in Go
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **CRD (Custom Resource Definition)** | The schema that registers a brand-new API resource type with Kubernetes |
| **CR (Custom Resource)** | An actual instance/object created against a CRD's schema |
| **Custom Controller** | An application that watches Custom Resources and executes logic to reconcile actual state with desired state |
| **client-go** | Kubernetes' official Go client library for interacting with the API Server |
| **Controller Runtime** | A framework providing pre-built Watchers, Informers, and reconciliation scaffolding |
| **Watcher** | Subscribes to change events (Create/Update/Delete) for a given resource type |
| **Informer** | Caches resource state locally to reduce direct API Server load |
| **Worker Queue** | Holds keys of changed objects awaiting processing, enabling retries and decoupled handling |
| **Reconciliation** | The loop of comparing actual vs. desired state and taking action to close the gap |
| **Operator Pattern** | The general term for the CRD + Controller combination used to automate operational logic |
| **CNCF** | Cloud Native Computing Foundation — hosts and standardizes major cloud-native projects like Istio, Argo CD, Prometheus |
| **Kubernetes Sample Controller** | Official minimal reference repo demonstrating the full CRD + Controller pattern |

---

## 📂 Summary of Tasks
- ✅ Understood: Why native Kubernetes resources fall short for tools like Istio, Argo CD, and Keycloak.
- ✅ Learned: The three-piece extensibility model — CRD (schema), CR (instance), Controller (logic).
- ✅ Understood: A CRD alone only registers a type; a CR alone is inert data; the Controller is what makes anything actually happen.
- ✅ Learned: Why Golang + client-go is the standard for writing controllers.
- ✅ Understood: How the Controller Runtime's Watchers/Informers feed into a worker queue for reconciliation.
- ✅ Walked through: The full Create/Update/Delete → Watch → Queue → Reconcile workflow.
- ✅ Understood: Why Helm is the standard way to deploy CRDs and Controllers together in production.
- ✅ Practiced (conceptually): Debugging a custom controller via `kubectl logs` and `kubectl describe`.
- ✅ Noted: CNCF and the Kubernetes Sample Controller repo as reference points for real-world patterns.

---

## 💡 My Takeaway

The distinction that mattered most today: a Custom Resource Definition doesn't create behavior — it creates *vocabulary*. It teaches the API Server a new noun. The actual verb — the thing that reacts and does work — is entirely the Controller's job, running its own watch loop completely separate from the API Server itself. That reframes tools like Istio or Argo CD less as "Kubernetes features" and more as "applications that happen to extend Kubernetes' API and then actively work to keep reality matching whatever CRs describe."

The worker queue detail was a good one to internalize — it's the same pattern I've now seen in a few different contexts (this, and general event-driven system design): don't act directly inside the event handler, decouple detection from processing so retries and backpressure are possible. Seeing that pattern show up inside Kubernetes' own controller runtime made it click as a general distributed-systems idiom, not just a Kubernetes quirk.

The practical framing at the end was the most useful for where I actually am right now — most DevOps work with CRDs isn't writing new controllers from scratch, it's debugging existing ones (Argo CD, cert-manager, Istio) when reconciliation gets stuck. Knowing to check controller logs and `describe` the CR itself, rather than guessing blindly, is the actual day-to-day skill here.

---

## 📈 Next Up
**Day 29:** Writing a minimal Custom Controller in Go using client-go and the Controller Runtime — working through the Kubernetes Sample Controller repository hands-on.

---

## 🔗 Resources
* [Kubernetes CRD Documentation](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* [Kubernetes Sample Controller Repository](https://github.com/kubernetes/sample-controller)
* [client-go Library](https://github.com/kubernetes/client-go)
* [Kubernetes Controller Runtime](https://github.com/kubernetes-sigs/controller-runtime)
* [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/)
* [Argo CD](https://argo-cd.readthedocs.io/) · [Istio](https://istio.io/) · [Prometheus Operator](https://prometheus-operator.dev/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*