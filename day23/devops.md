
![Progress](https://img.shields.io/badge/Progress-23%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 23 — Kubernetes Services Deep Dive (with KubeShark)

## 📝 Topic: Load Balancing, Service Discovery & Visualizing Traffic with KubeShark
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 30, 2026

---

## 🎯 Learning Objectives
* Understand why relying on dynamic Pod IPs is unreliable in practice, not just in theory.
* Practically expose an application using NodePort.
* Understand how LoadBalancer Services interact with the Cloud Controller Manager.
* Demonstrate Service Discovery via labels and selectors — and observe what happens when it breaks.
* Use KubeShark to visualize real traffic flow inside a Kubernetes cluster.
* Observe round-robin load balancing across multiple Pod replicas in real time.

---

## ❓ Part 1 — Why Pod IPs Are Unreliable

This session moves from theory to practice — actually watching the problem happen, not just reading about it.

```
Demo setup:
  Deployment with 2 replicas
  Pod A: IP 172.17.0.5
  Pod B: IP 172.17.0.7

A user bookmarks/hardcodes: http://172.17.0.5:8080
```

**What breaks this:**

```
Scenario 1 — Pod restart:
  Pod A crashes → ReplicaSet creates a new pod
  New pod gets a DIFFERENT IP: 172.17.0.9
  → The hardcoded 172.17.0.5 now points to nothing

Scenario 2 — Scaling:
  Deployment scales from 2 → 5 replicas
  → 3 new pods, 3 new IPs
  → A hardcoded IP only ever reaches ONE pod, never the other 4
  → No load distribution at all

Scenario 3 — Rescheduling:
  Node hosting Pod A fails
  → Pod A is rescheduled to a different node entirely
  → New IP, possibly different subnet range
```

> **The core lesson demonstrated live:** Pod IPs are an implementation detail, not a stable interface. Anything that depends on a Pod IP directly will break the moment Kubernetes does what it's designed to do — heal, scale, or reschedule.

This is exactly why Services exist — they provide a **stable abstraction layer** over an inherently unstable set of IPs.

---

## 🚪 Part 2 — Exposing Applications: NodePort (Hands-On)

### The Practical Demo

```bash
# Start a local cluster
minikube start

# Deploy an application with 2 replicas
kubectl apply -f deployment.yaml

kubectl get pods -o wide
# NAME                     READY   STATUS    IP            NODE
# my-app-7d7f4b5c9-2xkpq  1/1     Running   172.17.0.5    minikube
# my-app-7d7f4b5c9-8lmnt  1/1     Running   172.17.0.7    minikube
```

### Creating the NodePort Service

```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

```bash
kubectl apply -f nodeport-service.yaml

kubectl get svc
# NAME               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)
# my-app-nodeport    NodePort   10.96.45.123   <none>        80:30080/TCP
```

### Accessing the Application

```bash
# Get the Minikube node's IP
minikube ip
# 192.168.49.2

# Access via NodeIP:NodePort
curl http://192.168.49.2:30080
```

```
Traffic path:
  User → 192.168.49.2:30080 (Node IP : NodePort)
       → Kube-proxy intercepts via iptables rule
       → Routes to Service ClusterIP (10.96.45.123)
       → Service routes to one of the backing Pods
```

> **What's demonstrated:** No matter which pod handles the request, no matter how many times pods restart, the same `NodeIP:NodePort` combination keeps working. That stability is the entire point.

---

## ☁️ Part 3 — LoadBalancer Service Type

### How Cloud Provider Integration Works

```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

```bash
kubectl apply -f loadbalancer-service.yaml

kubectl get svc my-app-lb
# NAME         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)
# my-app-lb    LoadBalancer   10.96.45.124   <pending>        80:31234/TCP
```

**On a real cloud cluster (EKS/AKS/GKE), what happens behind the scenes:**

```
kubectl apply -f loadbalancer-service.yaml
        ↓
Kubernetes API Server records the request
        ↓
Cloud Controller Manager (CCM) detects: "new LoadBalancer Service"
        ↓
CCM calls the cloud provider's API (e.g. AWS ELB API)
        ↓
AWS provisions a real Elastic Load Balancer
        ↓
CCM updates the Service with the assigned public IP
        ↓
kubectl get svc now shows EXTERNAL-IP: 54.210.xxx.xxx
```

> **Important caveat shown in the demo:** On Minikube, `EXTERNAL-IP` stays `<pending>` forever because there's no real cloud provider to provision a load balancer. This is exactly why local clusters use NodePort for testing instead — LoadBalancer only fully works on managed cloud clusters.

```bash
# On Minikube, simulate LoadBalancer behavior
minikube tunnel
# Creates a route so LoadBalancer services get a reachable IP locally
```

---

## 🏷️ Part 4 — Service Discovery: Labels & Selectors in Action

### The Working State

```yaml
# Deployment pods have this label
metadata:
  labels:
    app: payment

# Service selector matches it
spec:
  selector:
    app: payment
```

```bash
kubectl get endpoints my-app-service
# NAME              ENDPOINTS
# my-app-service    172.17.0.5:8080,172.17.0.7:8080
```

The **Endpoints object** is the live, real-time list of Pod IPs a Service is currently routing to — built automatically from the label match.

### Live Demo: Breaking Service Discovery on Purpose

```bash
# Edit the service and CHANGE the selector label
kubectl edit svc my-app-service
```

```yaml
spec:
  selector:
    app: payment-v2    # changed from "payment" to "payment-v2"
```

**Immediate result:**

```bash
kubectl get endpoints my-app-service
# NAME              ENDPOINTS
# my-app-service    <none>          ← instant loss of all routing!

curl http://my-app-service
# curl: (7) Failed to connect — connection refused
```

> **What this demonstrates live:** The Service has zero awareness of "the app." It only knows "find pods with this exact label." Change the label by even one character and every pod silently becomes unreachable through that service — even though the pods themselves are still healthy and running.

```bash
# Revert the label to restore connectivity
kubectl edit svc my-app-service
# selector: app: payment   ← back to original

kubectl get endpoints my-app-service
# my-app-service    172.17.0.5:8080,172.17.0.7:8080   ← instantly restored
```

This is an important debugging lesson: **if a Service "stops working" with no errors anywhere, check the label selector first.**

---

## 🦈 Part 5 — Load Balancing Visualized with KubeShark

### What is KubeShark?

> KubeShark is a traffic analysis and packet inspection tool built specifically for Kubernetes — it shows live network traffic flowing through the cluster at Layer 4 and Layer 7.

```bash
# Install KubeShark
sh -c "$(curl -sL https://kubeshark.co/install)"

# Launch the dashboard
kubeshark tap

# Opens a web UI showing real-time traffic
```

### The Live Demo Setup

```
Deployment: my-app, 2 replicas
  Pod A: 172.17.0.5
  Pod B: 172.17.0.7

Service: my-app-service (ClusterIP)
  selector: app: my-app
```

### Sending Multiple Requests and Observing Round-Robin

```bash
# Send 10 requests in a loop
for i in {1..10}; do
  curl http://my-app-service
done
```

**What KubeShark visualizes in real time:**

```
Request 1  → routed to 172.17.0.5  (Pod A)
Request 2  → routed to 172.17.0.7  (Pod B)
Request 3  → routed to 172.17.0.5  (Pod A)
Request 4  → routed to 172.17.0.7  (Pod B)
Request 5  → routed to 172.17.0.5  (Pod A)
Request 6  → routed to 172.17.0.7  (Pod B)
...

Pattern: alternating, evenly distributed → ROUND-ROBIN
```

### Why This Matters

```
Without watching real traffic:
  "Services load balance" is a sentence you memorize

With KubeShark:
  "Services load balance" is something you SEE happening —
  packet by packet, request by request, alternating between
  172.17.0.5 and 172.17.0.7 in real time
```

> **The instructor's emphasis:** Every DevOps engineer should be comfortable using a traffic visualization tool like KubeShark. When a production issue is "intermittent" — works sometimes, fails other times — it's very often one specific pod behind a Service that's unhealthy. Without visibility into which pod is handling which request, this kind of issue is extremely hard to diagnose from logs alone.

---

## 🔍 Part 6 — KubeShark for Real Debugging

### Practical Use Cases

```
Scenario: "The API is slow sometimes but not always"
  → Use KubeShark to see which Pod is handling slow requests
  → Often reveals: Pod A is healthy, Pod B is the slow one
  → Without traffic visibility, you'd be debugging logs from both pods blindly

Scenario: "A specific user reports errors, others don't"
  → Use KubeShark to trace which pod handled that user's request
  → Correlate with pod-specific logs or resource usage

Scenario: "Is round-robin actually happening?"
  → Visually confirm load balancing is distributing evenly
  → Catch cases where one pod gets disproportionate traffic
    (could indicate connection pooling or session affinity issues)
```

### Layer 4 vs Layer 7 Visibility

```
Layer 4 (Transport):
  → TCP connections, source/destination IPs and ports
  → "Which pod is this connection going to?"

Layer 7 (Application):
  → HTTP headers, request paths, response codes
  → "What request was made? What response came back? How long did it take?"

KubeShark shows both — making it more powerful than basic
network tools that only see Layer 4.
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Pod IP** | The internal IP assigned to a Pod — unstable, changes on restart/reschedule |
| **NodeIP** | The IP address of a worker node in the cluster |
| **NodePort** | A Service type exposing the app on a fixed port across every node |
| **`minikube ip`** | Returns the IP address of the local Minikube node |
| **`minikube tunnel`** | Simulates LoadBalancer functionality on a local cluster |
| **Endpoints** | A Kubernetes object listing the live Pod IPs a Service currently routes to |
| **`kubectl get endpoints`** | Shows the real-time list of pods backing a Service |
| **Service Discovery** | The mechanism by which Services find Pods — via label/selector matching |
| **Round-Robin** | A load balancing algorithm that distributes requests evenly across all targets in sequence |
| **KubeShark** | A traffic inspection tool for visualizing live network flow inside a Kubernetes cluster |
| **Layer 4 (Transport)** | Network layer dealing with TCP/UDP connections, IPs, and ports |
| **Layer 7 (Application)** | Network layer dealing with HTTP requests, headers, and application-level data |
| **Packet Inspection** | Examining individual network packets to understand traffic behavior |
| **Cloud Controller Manager (CCM)** | The component that provisions real cloud load balancers for `type: LoadBalancer` Services |
| **`<pending>` EXTERNAL-IP** | Indicates a LoadBalancer Service has no cloud provider to fulfill the request (common on Minikube) |
| **Session Affinity** | A setting that routes a client's requests to the same pod repeatedly, instead of round-robin |

---

## 📂 Summary of Tasks
- ✅ Observed: Why dynamic Pod IPs break under restart, scaling, and rescheduling.
- ✅ Deployed: A NodePort Service and accessed an app via `NodeIP:NodePort`.
- ✅ Understood: How LoadBalancer Services interact with the Cloud Controller Manager.
- ✅ Observed: `EXTERNAL-IP: <pending>` on Minikube and why — no real cloud provider available.
- ✅ Demonstrated: Service Discovery working correctly via matching labels/selectors.
- ✅ Broke (intentionally): Service Discovery by changing a selector label — watched Endpoints go empty.
- ✅ Installed and used: KubeShark to visualize live traffic inside the cluster.
- ✅ Confirmed: Round-robin load balancing across 2 pod replicas, request by request.
- ✅ Understood: Layer 4 vs Layer 7 traffic visibility and real-world debugging use cases.

---

## 💡 My Takeaway

Watching Service Discovery break on purpose was more valuable than any explanation of how it works. Changing one character in a selector label — `payment` to `payment-v2` — and watching `kubectl get endpoints` go from two healthy pod IPs to `<none>` instantly is the kind of thing that builds real debugging instinct. The lesson that sticks: if a Service silently stops routing traffic and the pods themselves look perfectly healthy, the very first thing to check is whether the selector still matches the pod labels. No errors, no crash, no logs — just a label mismatch.

KubeShark reframed load balancing from a concept I'd accept on faith to something I watched happen, request by request, alternating cleanly between two pod IPs. That distinction matters more than it sounds — "I know Services load balance" and "I have watched a Service load balance and could debug it if it stopped" are very different levels of understanding, and the second one is what actually matters when something breaks in production at 2am.

---

## 📈 Next Up
**Day 26:** Kubernetes Ingress — routing external HTTP/HTTPS traffic to multiple services based on path and host, replacing the need for a LoadBalancer per service.

---

## 🔗 Resources
* [KubeShark Official Site](https://kubeshark.co/)
* [Kubernetes Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Kubernetes Endpoints](https://kubernetes.io/docs/concepts/services-networking/service/#endpoints)
* [Minikube Tunnel Docs](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel)
* [Debugging Services — Official K8s Docs](https://kubernetes.io/docs/tasks/debug/debug-application/debug-service/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
