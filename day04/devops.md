![Progress](https://img.shields.io/badge/Progress-4%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 04  — DevOps in the Real World + Virtual Machines

## 📝 Topic: Org Roles, SDLC, Jira & Virtualization Fundamentals
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 8, 2026

---

## 🎯 Learning Objectives
* Understand how requirements flow from customers to a DevOps engineer through organizational layers.
* Map every key role in a product team and know what each one owns.
* Understand where DevOps engineers plug into the SDLC.
* Use Jira terminology correctly — Epics, Stories, Sprints, Blockers.
* Explain what a physical server is and why it's inefficient on its own.
* Define Virtual Machines and understand how a Hypervisor creates them.
* Understand how AWS and cloud providers use virtualization at scale.

---

## 🏢 Part 1 — How a Real Organization Works (Amazon Fresh Example)

Requirements don't land on a DevOps engineer's desk directly from customers. They pass through several layers before reaching the engineering team.

### 🔄 The Requirement Flow

```
Customer
   ↓
Business Analyst (BA)        ← gathers requirements, writes BRD
   ↓
Product Manager (PM)         ← defines product vision, prioritizes
   ↓
Product Owner (PO)           ← breaks into Epics and Stories
   ↓
Solutions Architect          ← defines HLD and LLD
   ↓
Development / Scrum Team     ← Dev, QE, DBA, DevOps — builds in sprints
   ↓
SRE & Technical Writers      ← maintains reliability, writes docs
```

---

### 👥 Key Roles Breakdown

#### 📋 Business Analyst (BA)
* The bridge between the customer and the tech team
* Gathers requirements directly from customers through interviews and workshops
* Produces the **BRD (Business Requirement Document)** — the formal record of what the customer wants
* Example: A customer says "I want 15-minute delivery in my city." The BA documents this as a requirement.

#### 🎯 Product Manager (PM)
* Owns the **product vision** — what the product should become long-term
* Prioritizes which requirements get built first based on business value
* Decides what goes into the current release vs future releases

#### 📌 Product Owner (PO)
* Takes the PM's priorities and breaks them into **Epics** and **Stories**
* Works directly with the Scrum team during sprint planning
* Owns the **product backlog** — the ordered list of all work to be done
* Example: Epic = "15-minute delivery service." Stories = "Build location tracking API," "Set up delivery routing DB," etc.

#### 🏗️ Solutions Architect
* Defines the **technical blueprint** for implementing requirements
* Produces two documents:
  * **HLD (High-Level Design)** — overall system architecture, how services connect
  * **LLD (Low-Level Design)** — module-level logic, data flows, database schemas
* DevOps engineers read the HLD/LLD to understand what infrastructure to provision

#### 💻 Development / Scrum Team
The core delivery unit. Works in **sprints** (typically 2-week cycles):

| Role | Responsibility |
|---|---|
| **Developer** | Writes application code |
| **QE (Quality Engineer)** | Tests and validates the code |
| **DBA (Database Admin)** | Manages databases and queries |
| **DevOps Engineer** | Provides infra, pipelines, Kubernetes, Docker |

DevOps engineers are **embedded in the Scrum team** — not a separate department.

#### 🛡️ SRE (Site Reliability Engineer)
* Responsible for **post-deployment reliability** — keeping systems running
* Owns monitoring, alerting, incident response, and on-call rotations
* Bridges the gap between development velocity and operational stability

#### 📝 Technical Writers
* Document the system — APIs, runbooks, architecture decisions
* Ensure institutional knowledge doesn't live only in engineers' heads

---

## 🔄 Part 2 — DevOps in the SDLC

DevOps engineers contribute at specific stages of the SDLC rather than owning all of it:

| SDLC Phase | DevOps Contribution |
|---|---|
| **Planning** | Estimate infrastructure needs from HLD/LLD |
| **Building** | Provide Kubernetes clusters, Dockerfiles, dev environments |
| **Testing** | Set up automated test pipelines (CI) |
| **Deployment** | Manage CD pipelines, release automation, rollback mechanisms |
| **Monitoring** | Set up observability — logs, metrics, alerts |

### 🛠️ The Core DevOps Toolkit at This Stage

```
Kubernetes clusters   →  container orchestration for the application
Dockerfiles           →  consistent, reproducible build environments
CI/CD pipelines       →  automate test → build → deploy on every commit
```

> **The DevOps goal:** Make the path from developer's code to production as fast, reliable, and automated as possible.

---

## 📋 Part 3 — Project Management with Jira

Jira is the industry-standard tool for tracking work across a product team. Every requirement — from a customer's idea to a deployed feature — is tracked as a Jira item.

### 🗂️ The Jira Hierarchy

```
Epic
 └── Story
      └── Sub-task (optional)
```

### 📌 Epics
* High-level goals that may take multiple sprints to complete
* Created by the **Product Owner**
* Example: *"Build 15-minute delivery service for Tier-1 cities"*

### 📝 Stories
* Small, actionable tasks that fit within a single sprint (2 weeks)
* Assigned to specific team members during **sprint planning**
* Written from a user's perspective: *"As a [user], I want [feature] so that [benefit]"*
* Example stories under the delivery Epic:
  * "Set up Kubernetes cluster for delivery microservice" → DevOps
  * "Build location tracking API" → Backend Dev
  * "Write integration tests for routing logic" → QE

### 📊 How DevOps Engineers Use Jira

```
Sprint starts
   ↓
PO assigns Stories to DevOps engineer
   ↓
DevOps picks up Story: "Set up infra for delivery service"
   ↓
Updates status: To Do → In Progress → Done
   ↓
Logs blockers if any (e.g., "waiting for AWS quota increase")
   ↓
Management monitors progress and unblocks
```

> **Why this matters:** Without Jira, management has no visibility. A DevOps engineer who updates their Stories consistently is one that management can trust and defend in stakeholder meetings.

### 🔑 Jira Terms to Know

| Term | What it means |
|---|---|
| **Epic** | A large, high-level goal spanning multiple sprints |
| **Story** | A small, deliverable task within one sprint |
| **Sprint** | A fixed time-box (usually 2 weeks) for delivering a set of Stories |
| **Backlog** | The ordered list of all pending Stories and Epics |
| **Blocker** | An impediment preventing a Story from progressing |
| **Story Points** | A relative estimate of effort — not hours |
| **Sprint Planning** | The ceremony where the team pulls Stories from the backlog into the sprint |
| **Sprint Review** | Demo of completed work at the end of a sprint |
| **Retrospective** | Team reflection on what went well and what to improve |

---

## 🖥️ Part 4 — Virtual Machines & Virtualization

### ❓ What is a Server?

> *"A server is a powerful computer system used to host and deploy applications, making them accessible to users over the internet."*

A server is just a computer — but purpose-built for:
* High uptime (24/7 operation)
* More RAM, CPU, and storage than a laptop
* Network connectivity for serving requests from users worldwide

### ⚠️ The Problem: Physical Servers Are Wasteful

**The land analogy:**

Imagine you own 10 acres of land. You use only 1 acre for farming. The other 9 acres sit empty — no rent, no crop, no return.

Physical servers have the same problem:

```
Physical server capacity:  16 CPU cores, 128 GB RAM
One application uses:       2 CPU cores,  8 GB RAM
Wasted:                     14 CPU cores, 120 GB RAM  ← idle 24/7
```

In the pre-virtualization era, one server = one application. The rest of the hardware sat idle.

### 💡 The Solution: Virtual Machines

> *"A VM is a logical partition of a physical server. It allows multiple isolated environments to run on a single piece of hardware, each with its own dedicated memory, CPU, and resources."*

```
┌─────────────────────────────────────────────┐
│           Physical Server                   │
│   16 Cores │ 128 GB RAM │ 2 TB Storage      │
│                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │  VM 1   │ │  VM 2   │ │  VM 3   │       │
│  │ 4 cores │ │ 4 cores │ │ 4 cores │       │
│  │ 32 GB   │ │ 32 GB   │ │ 32 GB   │       │
│  │ App A   │ │ App B   │ │ App C   │       │
│  └─────────┘ └─────────┘ └─────────┘       │
│                                             │
│           [ Hypervisor ]                    │
└─────────────────────────────────────────────┘
```

Each VM is fully isolated — App A cannot see or affect App B's memory.

### ⚙️ The Hypervisor

> *"The hypervisor is the software responsible for creating and managing logical partitions, enabling multiple VMs to run on a single physical host."*

The hypervisor sits between the hardware and the VMs:

```
Physical Hardware (CPU, RAM, Disk)
          ↕
      Hypervisor              ← creates & manages VMs
     ↙    ↓    ↘
  VM 1   VM 2   VM 3          ← each thinks it owns the hardware
```

**Two types of hypervisors:**

| Type | How it works | Examples |
|---|---|---|
| **Type 1 (Bare Metal)** | Runs directly on hardware — no OS underneath | VMware ESXi, Microsoft Hyper-V, AWS Nitro |
| **Type 2 (Hosted)** | Runs on top of an existing OS | VirtualBox, VMware Workstation |

Cloud providers like AWS use **Type 1** hypervisors for maximum performance and security.

### ☁️ How AWS Uses Virtualization

AWS data centers are warehouses full of physical servers. When you launch an EC2 instance:

```
You request: "Give me a t3.medium instance in us-east-1"
                    ↓
AWS hypervisor carves out a VM on a physical server in Virginia
                    ↓
You get: 2 vCPUs, 4 GB RAM — fully isolated, yours to use
                    ↓
You pay only for what you use — the hour you run it
```

**Why AWS has data centers in multiple regions:**

```
User in Mumbai → connects to AWS ap-south-1 (Mumbai)   → ~10ms latency
User in Mumbai → connects to AWS us-east-1 (Virginia)  → ~200ms latency
```

Regional data centers reduce latency by putting compute closer to the user.

---

## 🔗 Part 5 — Why Virtualization is Core to DevOps

| DevOps Pillar | How Virtualization Supports It |
|---|---|
| **Efficiency** | One physical server runs many isolated environments |
| **Speed** | Spin up a new VM in seconds — no hardware procurement |
| **Consistency** | Every VM is identical — no "works on my machine" |
| **Scalability** | Add more VMs instantly when traffic spikes |
| **Cost** | Pay per VM-hour instead of buying dedicated hardware |

> **The shift:** From underutilizing expensive physical hardware → running multiple virtual environments simultaneously, maximizing every CPU core and GB of RAM.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **BRD** | Business Requirement Document — formal record of customer requirements |
| **HLD** | High-Level Design — system-wide architecture |
| **LLD** | Low-Level Design — module-level implementation details |
| **Epic** | A large Jira item representing a high-level goal spanning multiple sprints |
| **Story** | A small Jira item representing an actionable task within one sprint |
| **Sprint** | A fixed time-box (usually 2 weeks) for delivering work |
| **Backlog** | The ordered list of all pending work |
| **Scrum Team** | The cross-functional team (Dev, QE, DBA, DevOps) that delivers sprints |
| **SRE** | Site Reliability Engineer — owns uptime, monitoring, and incident response |
| **CI/CD** | Continuous Integration / Continuous Deployment — automates build, test, deploy |
| **Server** | A powerful computer that hosts applications and serves users over the internet |
| **Virtual Machine (VM)** | A logical partition of a physical server with dedicated CPU, RAM, and storage |
| **Hypervisor** | Software that creates and manages VMs on physical hardware |
| **Type 1 Hypervisor** | Runs directly on bare metal — used by AWS, VMware ESXi |
| **Type 2 Hypervisor** | Runs on top of an OS — used for local dev, e.g. VirtualBox |
| **EC2** | AWS Elastic Compute Cloud — the service that provides virtual machines on AWS |
| **Region** | A geographic cluster of AWS data centers |
| **Latency** | The delay between a user's request and the server's response |
| **Resource Utilization** | The percentage of available compute resources actually being used |

---

## 📂 Summary of Tasks
- [x] Understood: How requirements flow from customer → BA → PM → PO → Architect → Dev Team.
- [x] Understood: Every role in a product org and what they own.
- [x] Understood: Where DevOps engineers fit in the SDLC — infra, pipelines, and tooling.
- [x] Practiced: Jira terminology — Epic, Story, Sprint, Backlog, Blocker, Story Points.
- [x] Understood: Why physical servers are wasteful (the land analogy).
- [x] Understood: What a Virtual Machine is and how it solves the inefficiency problem.
- [x] Understood: The role of a Hypervisor — Type 1 vs Type 2.
- [x] Understood: How AWS uses virtualization — EC2, regions, and latency reduction.

---

## 💡 My Takeaway

Two things clarified a lot today:

**On org structure:** Before this, "DevOps" felt like a vague role floating somewhere between dev and ops. Now it has a precise location — embedded in the Scrum team, picking up Stories in Jira, providing the infrastructure the developers need to ship. The requirement flow from customer → BA → PM → PO → Architect → Team explains why DevOps engineers never talk directly to the customer — by the time work reaches them, it's been translated into technical tasks they can act on.

**On virtualization:** The land analogy is the right mental model. A physical server with one app is an 8-lane highway with one car on it. A hypervisor is what turns that highway into a logical grid where dozens of cars can travel in isolated lanes simultaneously. This is the foundation that everything in cloud computing — Kubernetes, EC2, containers — is built on.

---

## 📈 Next Up
**Day 06:** Linux Fundamentals — the operating system that runs inside every VM, every container, and every cloud server in the stack.

---

## 🔗 Resources
* [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
* [Jira Docs — Getting Started](https://www.atlassian.com/software/jira/guides/getting-started/introduction)
* [AWS EC2 Overview](https://aws.amazon.com/ec2/)
* [What is a Hypervisor? — Red Hat](https://www.redhat.com/en/topics/virtualization/what-is-a-hypervisor)
* [VMware: Type 1 vs Type 2 Hypervisors](https://www.vmware.com/topics/glossary/content/hypervisor.html)
* [Scrum Guide — Official](https://scrumguides.org/scrum-guide.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*