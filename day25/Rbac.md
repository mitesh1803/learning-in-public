![Progress](https://img.shields.io/badge/Progress-25%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 25 — Kubernetes RBAC (Role-Based Access Control)

## 📝 Topic: Identity, Permissions & Bindings — Securing Cluster Access
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 02, 2026

---

## 🎯 Learning Objectives
* Understand what RBAC is and why it's non-negotiable in production clusters.
* Learn the three core building blocks of Kubernetes access control: Identity, Permissions, Connection.
* Understand how Kubernetes delegates user authentication to external Identity Providers instead of managing its own user database.
* Understand Service Accounts as the identity mechanism for Pods.
* Distinguish between **Role** (namespace-scoped) and **ClusterRole** (cluster-wide) permissions.
* Understand how RoleBindings connect an identity to a set of permissions.
* Set up a free OpenShift Sandbox environment for hands-on practice in the next session.

---

## 🔐 Part 1 — Introduction to Kubernetes RBAC

### What RBAC Actually Is

```
RBAC = Role-Based Access Control

A security framework that answers one question, every single time
someone or something tries to do anything in the cluster:

  "Is THIS identity allowed to perform THIS action on THIS resource?"
```

### Why It Matters

```
Without RBAC (or with it misconfigured):
  → Any user/service can potentially read, modify, or DELETE
    critical cluster resources
  → Includes highly sensitive components like etcd
    (the cluster's entire state and secrets live here)

  A single overprivileged service account or careless kubectl
  command could take down an entire production cluster.
```

> **The core lesson:** RBAC isn't an optional hardening step — it's the mechanism that prevents both malicious access and honest human mistakes from becoming production incidents.

### The Three Core Components

```
1. Identity          →  Users & Service Accounts
                         "WHO is asking?"

2. Permissions        →  Roles & ClusterRoles
                         "WHAT are they allowed to do?"

3. Connection          →  RoleBindings & ClusterRoleBindings
                         "WHICH identity gets WHICH permissions?"
```

These three pieces always work together — an identity with no binding has no permissions, and a Role with no binding is never actually granted to anyone.

---

## 👤 Part 2 — User Management & Service Accounts

### Kubernetes Doesn't Manage Users Itself

```
Unlike a Linux server:
  Linux:       useradd, passwd → users stored locally on the machine
  Kubernetes:  NO local user database exists at all
```

Instead, Kubernetes **delegates authentication** entirely to external **Identity Providers (IdPs)**:

```
Common Identity Providers used with Kubernetes:
  • AWS IAM        (common on EKS)
  • Okta
  • LDAP
  • Keycloak
```

```
Flow:
  User attempts kubectl command
        ↓
  Kubernetes API Server checks: "Who signed this request?"
        ↓
  Authentication is verified against the configured IdP
        ↓
  If valid → request proceeds to RBAC's Authorization check
  If invalid → request rejected before RBAC is even evaluated
```

> **Key distinction:** Authentication ("who are you?") happens via the external IdP. RBAC governs Authorization ("what are you allowed to do?") — a separate step that happens after identity is confirmed.

### Service Accounts — Identity for Pods, Not People

```
Every Pod in Kubernetes runs AS a Service Account:
  → If none is specified, it automatically uses the "default"
    service account of that namespace
  → The Service Account is how the Pod authenticates itself
    when talking to the API Server
```

```
Example:
  A monitoring Pod needs to LIST pods across the cluster
  → It authenticates to the API Server using its Service Account
  → RBAC then decides if that Service Account is ALLOWED to list pods
```

> **Why this matters in practice:** Applications and controllers running inside the cluster (not humans) are constantly making API calls — their access needs to be scoped just as carefully as a human user's, often more so, since they run unattended.

---

## 🎭 Part 3 — Roles and Bindings

### Role vs. ClusterRole

| Resource | Scope | Example Use Case |
|---|---|---|
| **Role** | A single, specific **namespace** | "This service account can read Pods only in the `payments` namespace" |
| **ClusterRole** | The **entire cluster**, across all namespaces | "This user can list Nodes cluster-wide" (Nodes aren't namespaced, so this requires a ClusterRole) |

```yaml
# Example: Role (namespace-scoped)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: payments
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

```yaml
# Example: ClusterRole (cluster-wide)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list"]
```

### RoleBindings — The Connecting Piece

```
A Role or ClusterRole, on its own, GRANTS NOTHING to anyone.
It's just a declared set of permissions sitting unused.

A RoleBinding (or ClusterRoleBinding) is what actually
CONNECTS an identity (User / Service Account) to that Role.
```

```yaml
# Example: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: payments
subjects:
  - kind: ServiceAccount
    name: monitoring-sa
    namespace: payments
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```
Without this binding:
  pod-reader Role exists → but grants access to NO ONE

With this binding:
  monitoring-sa Service Account → NOW can get/list/watch Pods
  in the payments namespace, and nowhere else
```

> **Mental model to remember:** Role = permission list on paper. RoleBinding = actually handing that paper to someone. Neither piece is useful without the other.

---

## ☁️ Part 4 — Hands-On Setup: OpenShift Sandbox

### Why OpenShift Sandbox for This Series

```
Free 30-day OpenShift Sandbox:
  → Gives a shared cluster with a PRIVATE namespace per user
  → No local cluster setup required
  → Close enough to a real production-style environment
    to practice RBAC concepts meaningfully
```

### What Was Set Up Today

```
1. Signed up for the free OpenShift Sandbox trial
2. Received a private namespace on a shared cluster
3. Extracted a login token from the OpenShift web console
4. Used that token to authenticate via CLI:
```

```bash
oc login --token=<extracted-token> --server=<cluster-api-url>

# or, since OpenShift is largely kubectl-compatible:
kubectl config use-context <sandbox-context>
```

```
5. Verified access by:
   - Creating a test Deployment
   - Scaling Pods up/down
   - Watching cluster Events to observe what's happening in real time
```

```bash
kubectl create deployment test-app --image=nginx
kubectl scale deployment test-app --replicas=3
kubectl get events --sort-by='.lastTimestamp'
```

> **Purpose of this setup:** This sandbox becomes the environment for the next session, where Service Accounts, Roles, and RoleBindings will actually be created and tested hands-on — rather than just described conceptually.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **RBAC** | Role-Based Access Control — governs what identities can do within a cluster |
| **Authentication** | Verifying WHO is making a request (handled by external IdPs, not Kubernetes itself) |
| **Authorization** | Verifying WHAT an already-authenticated identity is allowed to do (this is RBAC's job) |
| **Identity Provider (IdP)** | External system (AWS IAM, Okta, LDAP, Keycloak) that Kubernetes delegates user authentication to |
| **Service Account** | An identity assigned to a Pod for authenticating with the API Server; defaults to "default" if unspecified |
| **Role** | A namespace-scoped set of permissions |
| **ClusterRole** | A cluster-wide set of permissions, required for non-namespaced resources like Nodes |
| **RoleBinding** | Connects a Role to an identity within a specific namespace |
| **ClusterRoleBinding** | Connects a ClusterRole to an identity across the entire cluster |
| **etcd** | The cluster's backing key-value store holding all state — a primary reason RBAC exists, to protect this from unauthorized access |
| **OpenShift Sandbox** | A free, time-limited shared OpenShift cluster used here for hands-on RBAC practice |
| **`oc` CLI** | OpenShift's command-line tool, largely compatible with `kubectl` |

---

## 📂 Summary of Tasks
- ✅ Understood: RBAC's role in preventing unauthorized access and accidental destructive actions in production.
- ✅ Learned: The three-part model of Identity → Permissions → Connection.
- ✅ Understood: Kubernetes delegates authentication to external Identity Providers (AWS IAM, Okta, LDAP, Keycloak) rather than managing users itself.
- ✅ Understood: Service Accounts as the identity mechanism Pods use to authenticate with the API Server.
- ✅ Distinguished: Role (namespace-scoped) vs. ClusterRole (cluster-wide) permissions.
- ✅ Understood: RoleBindings/ClusterRoleBindings as the mechanism that actually grants defined permissions to an identity.
- ✅ Set up: A free 30-day OpenShift Sandbox with a private namespace.
- ✅ Extracted: A login token from the OpenShift console for CLI access.
- ✅ Practiced: Creating a deployment, scaling pods, and monitoring cluster events in the sandbox.

---

## 💡 My Takeaway

The mental model that stuck hardest today: a Role by itself grants access to **no one** — it's just a declared list of permissions sitting inert until a RoleBinding actually connects it to an identity. That three-piece split (Identity / Permissions / Connection) makes RBAC feel far less abstract than "Kubernetes has a permissions system" — it's really three separate, composable objects that only become meaningful together.

The bigger reframe was realizing Kubernetes doesn't manage users at all. Coming from a Linux `useradd` mental model, it's easy to assume there's some equivalent local user store — there isn't. Authentication is entirely offloaded to external IdPs like AWS IAM or Okta, and Kubernetes RBAC only ever handles the authorization layer *after* that identity is already verified. That's a clean separation of concerns, but it also means RBAC configuration alone is never the full security picture — the IdP integration matters just as much.

Setting up the OpenShift Sandbox today without touching RBAC objects yet was a smart sequencing choice — next session is where Roles, Service Accounts, and Bindings actually get created and tested against real permission boundaries, instead of staying theoretical.

---


## 🔗 Resources
* [Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [Kubernetes Authentication Concepts](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
* [Kubernetes Service Accounts](https://kubernetes.io/docs/concepts/security/service-accounts/)
* [Red Hat OpenShift Sandbox](https://developers.redhat.com/developer-sandbox)
* [OpenShift `oc` CLI Documentation](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)

---
