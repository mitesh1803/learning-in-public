![Progress](https://img.shields.io/badge/Progress-50%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 07 — Production-Grade AWS Architecture Project

## 📝 Topic: VPC + High Availability + ASG + ALB + Bastion Host, End to End

**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 10, 2026

---

## 🎯 Learning Objectives

- Understand how VPC, Subnets, ASG, ALB, NAT Gateway, and a Bastion Host combine into one production architecture.
- Build a VPC with public/private subnets spread across two Availability Zones.
- Create a Launch Template and Auto Scaling Group to maintain instances automatically.
- Deploy a Bastion Host as the sole entry point for administrative SSH access.
- Deploy an application on private instances and expose it safely via an Application Load Balancer.
- Troubleshoot the most common reachability failure: Load Balancer Security Group misconfiguration.

---

## 🏗️ Part 1 — Project Architecture Overview

### The Goal

```
Deploy an application SECURELY and SCALABLY using:

  VPC (Public + Private Subnets)  → isolate app from direct internet exposure
  High Availability (2 AZs)       → survive a single data center failure
  NAT Gateway                     → private instances get outbound internet, stay hidden
  Auto Scaling Group (ASG)        → automatically maintains the right instance count
  Application Load Balancer (ALB) → distributes traffic across healthy instances
  Bastion Host (Jump Server)      → the ONLY secure, auditable path for SSH into private instances
```

### The Full Picture

```
                         Internet
                            │
                    Internet Gateway
                            │
        ┌───────────────────┴───────────────────────┐
        │                  VPC                      │
        │                                           │
   ┌────┴──────┐  AZ-a               AZ-b   ┌───────┴────┐
   │  Public   │                            │  Public    │
   │  Subnet   │        ALB (spans both)    │  Subnet    │
   │           │◄──────────────────────────►│            │
   │ Bastion   │                            │            │
   │  Host     │                            │            │
   └────┬──────┘                            └───────┬────┘
        │                                           │
   ┌────┴──────┐                            ┌───────┴────┐
   │  Private  │                            │  Private   │
   │  Subnet   │                            │  Subnet    │
   │           │                            │            │
   │ EC2 (ASG) │◄──── NAT Gateway ─────────►│ EC2 (ASG)  │
   │ Instance 1│      (in public subnet)    │ Instance 2 │
   └───────────┘                            └────────────┘
```

> **Why every piece is there:** the public subnet exposes only the ALB and the Bastion Host to the internet. Everything that actually matters — the application instances — lives in private subnets, reachable only through the ALB (for user traffic) or the Bastion Host (for admin SSH access).

---

## 🛠️ Part 2 — Step-by-Step Implementation

### Step 1: VPC Creation

```
Configured:
  → 2 Public Subnets  (one per AZ)
  → 2 Private Subnets (one per AZ)
  → Spanning 2 Availability Zones total
```

> **Note flagged during the session:** ensure enough Elastic IPs are available in the target region before creating NAT Gateways — each NAT Gateway requires its own Elastic IP, and accounts have a default regional limit that can be hit unexpectedly when provisioning multiple NAT Gateways for a multi-AZ setup.

### Step 2: Launch Template & Auto Scaling Group

```
Launch Template:
  → AMI: Ubuntu
  → Defines instance type, key pair, Security Group,
    and any user-data bootstrap script for every
    instance the ASG creates

Auto Scaling Group:
  → Configured to maintain 2 running instances
  → One instance placed in EACH Availability Zone
    (not both in the same AZ) — this is what actually
    delivers the high-availability guarantee
```

> **Why this matters:** an ASG alone doesn't guarantee availability — it's the ASG _combined with_ subnet placement across two AZs that ensures a single AZ failure only takes out one instance, not both.

### Step 3: Bastion Host Setup

```
Deployed: in a PUBLIC subnet (needs to be internet-reachable
          for admins to SSH into it)
```

```bash
# Transfer the .pem key file to the Bastion host
scp -i local-key.pem local-key.pem ubuntu@<bastion-public-ip>:~/

# SSH into the Bastion host itself
ssh -i local-key.pem ubuntu@<bastion-public-ip>

# From INSIDE the Bastion host, SSH into a private instance
ssh -i local-key.pem ubuntu@<private-instance-ip>
```

> **The core security pattern here:** the private instances never get a public IP at all. The ONLY way to reach them via SSH is by first authenticating to the Bastion Host, then hopping from there — creating a single, auditable, tightly-controlled administrative entry point instead of exposing every instance to direct SSH from the internet.

### Step 4: Application Deployment

```bash
# On each private instance
python3 -m http.server 8000
```

> A deliberately simple app (a basic Python HTTP server) is used here so the focus stays entirely on the surrounding infrastructure — VPC, ASG, ALB, Bastion — rather than application complexity.

### Step 5: Load Balancer Configuration

```
Application Load Balancer (ALB):
  → Created in the PUBLIC subnets (spanning both AZs)
  → Attached to a Target Group pointing at the
    private instances on port 8000
```

```
Traffic flow:
  User → ALB (public subnet, port 80)
       → Target Group → forwards to private instances (port 8000)
```

### Troubleshooting: Load Balancer Unreachable

```
Common failure mode: ALB appears "healthy" but is unreachable
  from a browser

Root cause to check FIRST:
  → Does the ALB's Security Group allow INBOUND traffic
    on the listener port (e.g., port 80)?

  → This is the exact same "everything is configured except
    the one inbound firewall rule" pattern seen in every
    prior EC2 deployment project (static site, Jenkins) —
    just now happening at the Load Balancer level instead
    of the individual instance level.
```

---

## ✅ Part 3 — Key Takeaways

```
Security:
  → NEVER expose application servers directly to the internet
  → Use private subnets + a Load Balancer as the only path in

Resiliency:
  → Always deploy across AT LEAST two Availability Zones
  → A single-AZ deployment has no protection against
    a data-center-level outage, no matter how well everything
    else is configured

Management:
  → Use a Bastion Host for all administrative access
  → Follow the Principle of Least Privilege strictly —
    the Bastion Host itself should have the minimum
    Security Group rules necessary (SSH only, from known IPs)
```

---

## 📖 Key Terms

| Term                                | What it means                                                                                                |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Launch Template**                 | Defines the configuration (AMI, instance type, key pair, etc.) used to create instances in an ASG            |
| **Auto Scaling Group (ASG)**        | Automatically maintains a desired number of running instances, replacing failed ones and scaling with demand |
| **Application Load Balancer (ALB)** | Distributes incoming traffic across healthy instances in a Target Group                                      |
| **Target Group**                    | A set of instances an ALB routes traffic to, along with associated health check config                       |
| **Bastion Host (Jump Server)**      | A hardened instance in a public subnet used as the sole, auditable SSH entry point to private instances      |
| **NAT Gateway**                     | Allows private-subnet instances outbound-only internet access while masking their private IPs                |
| **Elastic IP**                      | A static public IP address; required (one per NAT Gateway) and subject to regional account limits            |
| **High Availability (HA)**          | Architecting resources across multiple Availability Zones to survive a single data center failure            |

---

## 📂 Summary of Tasks

- ✅ Understood: How VPC, ASG, ALB, NAT Gateway, and Bastion Host combine into one production architecture.
- ✅ Created: A VPC with 2 public and 2 private subnets across 2 Availability Zones.
- ✅ Noted: Elastic IP regional limits as a gotcha to check before provisioning multiple NAT Gateways.
- ✅ Created: A Launch Template (Ubuntu) and an ASG maintaining one instance per AZ.
- ✅ Deployed: A Bastion Host in the public subnet as the sole SSH entry point.
- ✅ Practiced: SCP-ing a `.pem` key to the Bastion, then SSH-hopping into private instances.
- ✅ Deployed: A simple Python HTTP server on the private instances.
- ✅ Configured: An ALB in the public subnets, attached to a Target Group targeting the private instances.
- ✅ Troubleshot: ALB unreachable issue traced to a missing inbound Security Group rule on the listener port.

---

## 💡 My Takeaway

This project is really the payoff for everything covered across AWS Days 1 through 6 — VPC, IAM, EC2, Security Groups/NACLs, and Route 53 all show up here as actual working pieces of one coherent architecture instead of isolated concepts. Seeing the Bastion Host pattern implemented end to end (SCP the key onto the Bastion, then SSH-hop from there into a private instance with no public IP at all) makes the "never expose application servers directly" principle concrete rather than just a slogan.

The ALB troubleshooting step was the fourth time in this series I've hit the exact same category of bug — something is fully deployed and correctly configured, but a single missing inbound Security Group rule makes it completely unreachable. At this point I'm treating "check the Security Group on the load balancer/listener first" as a permanent first step whenever something is "up" but not reachable, rather than rediscovering it project by project.

The Elastic IP limit note is a small but genuinely useful practical gotcha — it's the kind of thing that only shows up the first time you actually try to provision NAT Gateways across multiple AZs, and would otherwise cost real debugging time mid-deployment.

---

## 📈 Next Up

**Day 33:** Extending this architecture with HTTPS via ACM certificates on the ALB, and adding CloudWatch alarms tied to the Auto Scaling Group's scaling policies.

---

## 🔗 Resources

- [AWS Auto Scaling Groups Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-groups.html)
- [Application Load Balancer Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
- [Bastion Hosts on AWS](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/bastion-hosts.html)
- [NAT Gateways Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Elastic IP Addresses Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

---

_Follow my journey! Feel free to ⭐ this repository to stay updated._
