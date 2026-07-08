![Progress](https://img.shields.io/badge/Progress-30%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 02 — EC2 (Elastic Compute Cloud) Deep Dive

## 📝 Topic: EC2 Fundamentals, Instance Types, Regions/AZs & Hands-On Jenkins Deployment
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 08, 2026

---

## 🎯 Learning Objectives
* Break down what "EC2" actually stands for and what each word means in practice.
* Understand why EC2 reduces operational burden compared to managing physical hardware.
* Learn the major EC2 instance type families and when each is appropriate.
* Understand Regions and Availability Zones, and why they matter for fault tolerance.
* Deploy a real application (Jenkins) on an EC2 instance from scratch.
* Configure Security Groups to allow the correct inbound traffic for that application.

---

## 💻 Part 1 — Understanding EC2

### Breaking Down the Name

```
EC2 = Elastic Cloud Compute

Elastic  → Ability to scale resources (CPU, RAM, Disk) up or down on demand
Cloud    → Delivered via AWS's public cloud platform
Compute  → A virtual server/machine, made possible via hypervisor technology
```

> Essentially: EC2 is "rent a virtual machine, and resize it whenever you need to" — the direct practical application of the virtualization concept covered on AWS Day 01.

---

## ❓ Part 2 — Why Use EC2?

### Reduced Management Overhead

```
AWS handles:
  → Physical hardware maintenance
  → Hypervisor updates and patching
  → Underlying infrastructure security

Engineers instead focus on:
  → The application itself
  → Configuration and deployment
  → NOT racking servers or patching hypervisors
```

### Cost Efficiency

```
Pay-as-you-go model:
  → Billed for actual instance running time
  → Shut down instances during non-working hours
    (e.g., a dev/test environment overnight)
    to avoid paying for idle compute
```

> **Practical habit worth building now:** for anything that isn't a 24/7 production workload, stopping instances outside active hours is one of the simplest cost-saving practices available — no architectural change required, just discipline.

---

## 🖥️ Part 3 — Instance Types

| Family | Best suited for |
|---|---|
| **General Purpose** | Balanced CPU/memory/networking — standard applications (used for today's project) |
| **Compute Optimized** | High-performance computing, gaming servers, machine learning workloads |
| **Memory Optimized** | Real-time big data analytics, memory-intensive applications |
| **Storage Optimized** | Workloads requiring high disk throughput (large databases, data warehousing) |
| **Accelerated Compute** | Hardware-based acceleration (GPUs, FPGAs) for specialized workloads |

> **Selection principle:** the instance family should match the workload's actual bottleneck. A memory-intensive analytics job on a Compute Optimized instance wastes money on CPU it doesn't need while potentially starving on RAM — matching the family to the actual constraint avoids both under- and over-provisioning.

---

## 🌍 Part 4 — Regions and Availability Zones

### Regions

```
Geographic locations where AWS operates physical data centers.

Examples: Mumbai (ap-south-1), US East (us-east-1)
```

### Availability Zones (AZs)

```
Within a single Region, AWS maintains MULTIPLE
isolated data centers = Availability Zones

Region: Mumbai (ap-south-1)
  ├── AZ: ap-south-1a
  ├── AZ: ap-south-1b
  └── AZ: ap-south-1c
```

```
Why multiple AZs matter:
  If AZ-a experiences a failure (power outage, hardware issue, etc.):
    → Resources deployed only in AZ-a go down
    → Resources deployed across AZ-a AND AZ-b remain available

  → This is the foundation of high availability and fault tolerance
    on AWS: never put all resources in a single AZ if uptime matters
```

> **Direct connection to earlier learning:** this is the same principle as Kubernetes subnet isolation from the Networking Fundamentals session — contain blast radius by spreading critical resources across isolated boundaries, rather than concentrating everything in one place.

---

## 🛠️ Part 5 — Practical Project: Deploying Jenkins

### Step 1: Launching the Instance

```
AMI: Ubuntu
Instance Type: t2.micro (Free Tier eligible — 750 hours/month)
```

> `t2.micro` under the AWS Free Tier is specifically chosen here to keep this hands-on practice at zero cost, within the 750 free hours/month allowance.

### Step 2: Secure Connectivity via Key Pair

```
Key Pair (.pem file) used instead of a password:
  → Public/private key authentication
  → Far more secure than password-based SSH login
  → Downloaded once at instance creation — cannot be re-downloaded later
```

```bash
ssh -i jenkins-key.pem ubuntu@<instance-public-ip>
```

### Step 3: System Update and Java Installation

```bash
# Update package lists
sudo apt update

# Install Java (required — Jenkins runs on the JVM)
sudo apt install openjdk-17-jre -y
```

### Step 4: Installing Jenkins

```bash
# Add Jenkins repository key and source
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### Step 5: Security Group Configuration

```
Security Group inbound rule needed:
  Type: Custom TCP
  Port: 8080
  Source: My IP (or 0.0.0.0/0 for broader testing access)

Reason: Jenkins' default web UI listens on port 8080 —
without this inbound rule, the dashboard is completely
unreachable from a browser, regardless of Jenkins running correctly.
```

### Step 6: Verification

```
Browser → http://<instance-public-ip>:8080
→ Jenkins setup wizard loads, prompting for the initial admin password
```

```bash
# Retrieve the initial admin password from the instance
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

> **The recurring lesson from every deployment project so far:** the application (Jenkins) can be perfectly installed and running, but without the correct Security Group rule opening the right port, it remains completely inaccessible — this is the exact same "files in place but nothing listening/reachable" gap seen in the earlier static site EC2 deployment.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **EC2** | AWS's Elastic Compute Cloud — on-demand virtual servers |
| **Hypervisor** | The underlying technology enabling multiple virtual machines on shared physical hardware |
| **General Purpose Instance** | Balanced instance family suited for standard, everyday applications |
| **Compute Optimized Instance** | Instance family suited for CPU-intensive workloads (HPC, gaming, ML) |
| **Memory Optimized Instance** | Instance family suited for memory-heavy workloads (big data analytics) |
| **Storage Optimized Instance** | Instance family suited for high disk-throughput workloads |
| **Region** | A geographic location housing AWS data centers (e.g., Mumbai, US East) |
| **Availability Zone (AZ)** | An isolated data center within a Region, used for fault tolerance and high availability |
| **`t2.micro`** | A Free Tier-eligible EC2 instance type (750 hours/month free) |
| **Key Pair (`.pem`)** | Public/private key credentials used for secure SSH access instead of passwords |
| **Security Group** | AWS's virtual firewall controlling inbound/outbound traffic to an instance |
| **Jenkins** | An open-source automation server, commonly used for CI/CD pipelines |
| **Port 8080** | The default port Jenkins' web dashboard listens on |

---

## 📂 Summary of Tasks
- ✅ Broke down: What "Elastic Cloud Compute" actually means in practice.
- ✅ Understood: Why EC2 reduces operational burden — AWS manages hardware/hypervisor, engineers focus on the application.
- ✅ Learned: The five major EC2 instance type families and their ideal use cases.
- ✅ Understood: Regions vs. Availability Zones, and why spreading resources across AZs enables fault tolerance.
- ✅ Launched: An Ubuntu `t2.micro` EC2 instance (Free Tier eligible).
- ✅ Connected: Via SSH using a `.pem` key pair instead of password authentication.
- ✅ Installed: Java and Jenkins on the instance via `apt`.
- ✅ Configured: A Security Group rule allowing inbound traffic on port 8080.
- ✅ Verified: The Jenkins dashboard loading successfully via the instance's public IP.

---

## 💡 My Takeaway

The Security Group step at the end was another instance of a pattern I'm now seeing repeatedly across these deployment projects — Apache on the static site day, and now Jenkins today: getting the application installed and running correctly is necessary but genuinely not sufficient. Without the explicit inbound rule for the right port, none of the rest of the setup matters from an accessibility standpoint. I want to start treating "confirm the Security Group/firewall rule" as a standard, non-skippable checklist item on every deployment from now on, rather than something to debug reactively when a browser request times out.

The Regions/AZ section connected directly back to the subnet isolation principle from the Networking Fundamentals session — spreading resources across AZs to contain the blast radius of a single data center failure is architecturally the same idea as subnetting for security isolation. It's a good sign that these AWS Zero to Hero fundamentals are reinforcing rather than duplicating what's already been covered in the Kubernetes and networking tracks.

Instance type selection was also a useful framing to internalize early — matching the family to the actual bottleneck (CPU vs. memory vs. disk vs. specialized acceleration) rather than defaulting to "just pick something with more of everything" is the kind of judgment call that will matter directly when sizing instances for the Intervio and ATS Resume Shortlister projects later.

---


## 🔗 Resources
* [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
* [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
* [AWS Regions and Availability Zones](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)
* [AWS Free Tier](https://aws.amazon.com/free/)
* [Jenkins Installation Documentation](https://www.jenkins.io/doc/book/installing/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*