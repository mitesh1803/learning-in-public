![Progress](https://img.shields.io/badge/Progress-39%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 25 — AWS Load Balancers Deep Dive (ALB, NLB, GWLB)

## 📝 Topic: Why Load Balancers Exist, the OSI Model Connection, and Choosing the Right Type
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 19, 2026

---

## 🎯 Learning Objectives
* Understand why a single EC2 instance can't handle growing traffic alone.
* Recall the OSI model layers and understand why they matter for load balancer selection.
* Learn the Application Load Balancer's (ALB) Layer 7 capabilities and tradeoffs.
* Learn the Network Load Balancer's (NLB) Layer 4 capabilities and ideal use cases.
* Learn the Gateway Load Balancer's (GWLB) role for security appliances.
* Build a clear decision framework for choosing ALB vs. NLB vs. GWLB.

---

## ❓ Part 1 — Why Use a Load Balancer?

### Scalability & Availability

```
As user traffic GROWS (game servers, e-commerce, etc.):
  → A SINGLE EC2 instance eventually can't handle the load
  → Load balancers DISTRIBUTE requests across MULTIPLE
    instances to prevent downtime and slowness
```

> This is the exact motivation behind the ALB used in the Day 32.1 production architecture project and the Terraform-defined ALB from Day 24 — today's session finally gives that recurring component its own dedicated deep dive.

### Load Balancing Techniques

```
Round Robin:
  → Distributes traffic EQUALLY across all instances,
    regardless of their current load or capacity

Ratio/Weight-based:
  → Accounts for instance CAPACITY differences
  → e.g., a larger instance gets proportionally
    more traffic than a smaller one
```

---

## 🌐 Part 2 — OSI Model Overview (Recap)

```
Why this matters TODAY specifically: different load
balancer TYPES operate at different OSI layers —
understanding which layer determines what capabilities
(and limitations) each load balancer type actually has.
```

```
Layer 7 (Application)   → HTTP/HTTPS protocols
Layer 6 (Presentation)  → SSL/TLS encryption
Layer 5 (Session)        → Connection session management
Layer 4 (Transport)      → TCP/UDP, data segmented into packets
Layer 3 (Network)        → Routing through routers
Layer 2 (Data Link)      → Switches, MAC addresses
Layer 1 (Physical)       → Hardware cables, physical connections
```

> This is a direct recap of the full OSI walkthrough from the earlier Networking Fundamentals session — today's value is in applying that existing knowledge to a genuinely practical decision (which AWS load balancer type to choose) rather than learning the layers again from scratch.

---

## ⚖️ Part 3 — AWS Load Balancer Comparison

### Application Load Balancer (ALB)

```
Operates at: Layer 7 (Application)

Best for: HTTP/HTTPS traffic
```

```
Features:
  → ADVANCED ROUTING based on:
      - URL paths
      - Hostnames
      - Headers

  → This is EXACTLY the path-based/host-based routing
    capability first introduced conceptually back in
    the Kubernetes Ingress session — same routing
    IDEA, now available at the AWS load-balancer level
```

```
Trade-offs:
  → Can introduce SLIGHT LATENCY due to request
    inspection (Layer 7 requires actually reading
    the HTTP request content to route intelligently)
  → Generally MORE COSTLY than lower-layer alternatives
```

### Network Load Balancer (NLB)

```
Operates at: Layer 4 (Transport)

Best for: HIGH-PERFORMANCE use cases:
  → Game servers
  → Video streaming (e.g., YouTube-scale traffic)
```

```
Features:
  → Handles MILLIONS of requests per second
  → EXTREMELY LOW latency
  → Supports STICKY SESSIONS
    (essential for long-duration streams — a user's
     connection needs to consistently reach the SAME
     backend instance throughout a stream, not get
     rerouted mid-session)
```

> **Why NLB is faster than ALB:** operating at Layer 4 means it routes based purely on IP/port information without inspecting the actual HTTP request content — less work per request translates directly into lower latency and higher throughput, at the cost of the content-aware routing intelligence ALB provides.

### Gateway Load Balancer (GWLB)

```
Best for: VIRTUAL APPLIANCES:
  → Firewalls
  → VPNs
```

```
Features:
  → Designed for HIGHLY SECURE, ENCRYPTED traffic
  → Handles traffic for third-party NETWORK
    VIRTUAL APPLIANCES specifically
  → Provides a TRANSPARENT, highly available
    interface for these security tools
```

> **GWLB's distinct role:** unlike ALB/NLB, which distribute traffic TO application backends, GWLB is specifically for routing traffic THROUGH security/network appliances (transparently inserting a firewall or VPN inspection point into the traffic path) before it continues to its actual destination.

---

## 📊 Part 4 — Decision Framework

| Load Balancer | OSI Layer | Best For | Key Strength |
|---|---|---|---|
| **ALB** | 7 (Application) | Web applications, APIs | Intelligent, content-aware routing (path/host/header) |
| **NLB** | 4 (Transport) | Gaming, streaming, extreme throughput | Ultra-low latency, millions of req/sec, sticky sessions |
| **GWLB** | N/A (appliance-focused) | Firewalls, VPNs, security appliances | Transparent traffic insertion for third-party security tools |

```
Quick decision heuristic:

  Need to route based on URL path/hostname/headers?
    → ALB

  Need maximum throughput/lowest latency, especially
  for long-lived connections (streaming, gaming)?
    → NLB

  Need to route traffic THROUGH a security/network
  appliance transparently?
    → GWLB
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Load Balancer** | Distributes incoming traffic across multiple backend instances to prevent overload |
| **Round Robin** | A load balancing technique distributing traffic equally across all targets |
| **Ratio/Weight-Based Balancing** | A technique accounting for differing instance capacities when distributing traffic |
| **Application Load Balancer (ALB)** | AWS's Layer 7 load balancer, supporting content-aware routing |
| **Network Load Balancer (NLB)** | AWS's Layer 4 load balancer, optimized for extreme throughput and low latency |
| **Gateway Load Balancer (GWLB)** | AWS's load balancer for transparently routing traffic through security/network appliances |
| **Sticky Sessions** | A mechanism ensuring a client's requests consistently reach the same backend instance |
| **Path-Based Routing** | Routing traffic based on URL path (ALB feature, conceptually shared with Kubernetes Ingress) |

---

## 📂 Summary of Tasks
- ✅ Understood: Why a single EC2 instance can't sustain growing traffic without load balancing.
- ✅ Learned: Round Robin vs. Ratio/Weight-based load balancing techniques.
- ✅ Recapped: The full OSI model, now applied to a practical AWS load balancer decision.
- ✅ Learned: ALB's Layer 7 content-aware routing (path/host/header) and its latency/cost tradeoffs.
- ✅ Learned: NLB's Layer 4 high-throughput, low-latency design and sticky session support.
- ✅ Learned: GWLB's specialized role routing traffic through firewalls and VPN appliances.
- ✅ Built: A clear decision framework for choosing the right load balancer type by use case.

---

## 💡 My Takeaway

Today finally connected two things that had been sitting somewhat separately in my mental model: the OSI layers from the earlier Networking Fundamentals session, and the ALB used repeatedly throughout this series (Day 32.1's production architecture, Day 24's Terraform project). Understanding that ALB's path/host/header routing is only possible BECAUSE it operates at Layer 7 — actually reading the HTTP request — while NLB's speed comes precisely from NOT doing that inspection, makes the ALB-vs-NLB tradeoff feel like a direct consequence of OSI layer choice, not an arbitrary feature list to memorize.

The ALB path/host-based routing capability being conceptually identical to Kubernetes Ingress routing was a genuinely satisfying connection — same underlying idea (route based on the request's content, not just its destination IP), implemented at two different layers of two different platforms (AWS's load balancer layer vs. Kubernetes' Ingress Controller layer).

GWLB was the one genuinely new concept today — I hadn't previously considered that a load balancer could be used to transparently route traffic THROUGH a security appliance rather than TO an application backend. Worth remembering as a distinct category entirely, not just "a third type of the same thing" as ALB/NLB.

---


## 🔗 Resources
* [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
* [Application Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
* [Network Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html)
* [Gateway Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/gateway/introduction.html)
* [OSI Model Explained (Cloudflare)](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-osi-model/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*