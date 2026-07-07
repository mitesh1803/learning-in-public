![Progress](https://img.shields.io/badge/Progress-3%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 01 — Cloud Fundamentals & AWS Account Setup

## 📝 Topic: Evolution of Cloud Infrastructure, Private vs. Public Cloud, Why AWS, and Account Creation
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 07, 2026

---

## 🎯 Learning Objectives
* Understand the historical progression from on-premises data centers to virtualization to the cloud.
* Understand the resource-underutilization problem virtualization was built to solve.
* Distinguish private cloud from public cloud, and the tradeoffs of each.
* Understand why AWS specifically is the recommended platform to learn first.
* Understand cloud repatriation as a minor counter-trend, and why it isn't the norm.
* Successfully create and verify a new AWS root account.

---

## 🏢 Part 1 — Evolution of Cloud Infrastructure

### Traditional On-Premises

```
Before cloud computing:
  Organizations had to BUY physical servers themselves
  → Build and maintain their own data centers
  → Handle hardware maintenance directly
  → Manage networking infrastructure in-house
  → Handle power management (cooling, backup power, etc.)
  → Provide physical security for the facility
```

> **The overhead wasn't just cost** — it was an entire operational discipline (facilities, electricians, security staff, hardware technicians) that had nothing to do with the actual software the organization wanted to run.

### The Underutilization Problem

```
Common real-world scenario:

  Organization buys a large, expensive server:
    Capacity: 100GB RAM
  
  Application actually deployed only needs:
    Usage: 1GB RAM

  Result: ~99% of the purchased capacity sits idle,
  fully paid for, doing nothing.
```

> This is the core inefficiency that made virtualization — and later, cloud computing — such an obvious improvement: paying for capacity that mostly goes unused is expensive by design in the on-premises model.

### Virtualization — The Turning Point

```
Virtualization allows MULTIPLE virtual servers
to run on a SINGLE physical machine:

  Physical Server (100GB RAM)
      ├── Virtual Server A (10GB) → App 1
      ├── Virtual Server B (20GB) → App 2
      ├── Virtual Server C (15GB) → App 3
      └── Virtual Server D (55GB) → App 4

  → Same physical hardware, dramatically better utilization
  → Each virtual server is isolated from the others
```

> Virtualization is the direct technical foundation that public cloud providers later built their entire business model on top of.

---

## ☁️ Part 2 — Private Cloud vs. Public Cloud

### Private Cloud

```
Infrastructure managed ENTIRELY within the organization's own boundaries:
  → The org's own internal team handles setup, scaling, maintenance
  → Full control over hardware and data location
  → Can be costly and operationally complex to run well
  → Essentially "on-premises, but virtualized internally"
```

### Public Cloud

```
Provided by companies like AWS, Azure, and GCP:
  → THE PROVIDER manages the data centers and global infrastructure
  → Users simply RENT resources (e.g., EC2 instances) on demand
  → Pay-as-you-go pricing model — pay only for what's actually used
```

| Model | Who manages infra | Cost model | Best fit |
|---|---|---|---|
| **Private Cloud** | Your own internal team | Large upfront + ongoing operational cost | Orgs with strict data-residency/compliance needs, or already-heavy infra investment |
| **Public Cloud** | The cloud provider (AWS/Azure/GCP) | Pay-as-you-go, scales with usage | Startups, mid-scale orgs, anyone wanting to avoid upfront hardware investment |

> **Why public cloud won for most use cases:** the pay-as-you-go model directly solves the underutilization problem from Part 1 — you're not paying for 100GB of capacity to use 1GB of it; you provision (and pay for) roughly what you actually need, and can scale up or down as demand changes.

---

## 🏆 Part 3 — Why Choose AWS Specifically?

### First-Mover Advantage

```
AWS pioneered the public cloud market
  → Longest track record
  → Largest existing market share of any provider
```

### Career Opportunities

```
Because so many companies already run on AWS:
  → Learning AWS specifically offers the widest range
    of cloud-related job opportunities
  → Skills transfer reasonably well to Azure/GCP later,
    but AWS has the deepest current job market demand
```

### Comprehensive Service Suite

```
AWS has grown from simple virtual machines (EC2)
to 200+ specialized services, including:
  → Databases            (RDS, DynamoDB, Aurora)
  → Networking           (VPC, Route 53, CloudFront)
  → Containers/Orchestration (ECS, EKS — managed Kubernetes)
  → And many more specialized domains
```

> **Why this matters for learning strategy:** starting with AWS means the concepts learned (VPCs, IAM, EC2, S3) map directly onto the platform where the largest number of real-world jobs and existing production systems actually run.

---

## 🔄 Part 4 — Addressing Cloud Repatriation

```
Cloud Repatriation = organizations moving WORKLOADS
                      BACK from public cloud to on-premises

Scale of this trend: roughly 1-2% of companies
  → A real but genuinely minor trend, not a mainstream reversal
```

```
Why most organizations still prefer public cloud despite this:
  → Physical data center cost and maintenance burden
    remains a significant barrier for most orgs
  → The operational simplicity of "rent what you need"
    still outweighs the cost savings of full ownership
    for the vast majority of use cases
```

> **Balanced takeaway:** repatriation is real for a small number of companies with very specific circumstances (extremely predictable, massive, steady-state workloads where owning hardware becomes cheaper long-term) — but it isn't evidence that public cloud is losing its dominant position overall.

---

## 🛠️ Part 5 — Practical Guide: AWS Account Setup

### Step-by-Step Process

```
1. Visit the AWS sign-in page
2. Choose to create a new account (this becomes the ROOT USER)
3. Provide verification details:
     - Valid email address
     - Phone number
     - Physical address
4. Provide a payment method:
     - Credit or debit card required
     - Used purely for IDENTITY VERIFICATION (anti-spam measure)
```

### The Card Verification Step

```
AWS performs a small, temporary transaction on the card
  → Purpose: confirm the card is real and belongs to a real person
  → This amount is typically refunded back to the user shortly after
  → This is NOT the start of a paid subscription —
    it's purely a verification mechanism
```

> **Important habit to build immediately after account creation:** the root user has unrestricted access to everything in the account. Best practice (to be covered in a later session) is to avoid using the root user for daily work entirely, and instead create a separate IAM user with only the necessary permissions — but that's a topic for its own dedicated session.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **On-Premises** | Infrastructure physically owned and operated by the organization itself |
| **Virtualization** | Technology allowing multiple isolated virtual servers to run on one physical machine |
| **Private Cloud** | Cloud-style infrastructure managed entirely within an organization's own boundaries |
| **Public Cloud** | Infrastructure owned/managed by a provider (AWS, Azure, GCP) and rented on demand |
| **Pay-as-you-go** | Pricing model where you pay only for the resources actually consumed |
| **EC2** | AWS's core virtual machine / compute service |
| **Cloud Repatriation** | The trend of moving workloads back from public cloud to on-premises infrastructure |
| **Root User** | The initial, unrestricted account created when signing up for AWS — should not be used for daily operational work |
| **IAM User** | A separate, permission-scoped identity created within an AWS account (to be covered in a future session) |

---

## 📂 Summary of Tasks
- ✅ Understood: The historical shift from on-premises data centers to virtualization to cloud computing.
- ✅ Understood: The resource-underutilization problem that made virtualization a necessary evolution.
- ✅ Distinguished: Private cloud (self-managed) vs. public cloud (provider-managed, pay-as-you-go).
- ✅ Learned: Why AWS is the recommended starting platform — first-mover advantage, job market demand, 200+ service breadth.
- ✅ Understood: Cloud repatriation as a real but minor (~1-2%) counter-trend, not a mainstream shift.
- ✅ Created: A new AWS root account with email, phone, and address verification.
- ✅ Completed: Payment method verification via AWS's small temporary card transaction.

---

## 💡 My Takeaway

The underutilization example — 100GB of capacity purchased to run an app that needs 1GB — is a genuinely useful mental anchor for *why* cloud computing exists at all, beyond just "it's trendy." It reframes the entire public cloud model as a direct fix for a very concrete, quantifiable inefficiency in the old on-premises approach, rather than an abstract industry shift.

The cloud repatriation section was a good moment of intellectual honesty in the material — acknowledging a real counter-trend (1-2% of companies) instead of presenting public cloud as having zero downsides. It's a useful habit to notice when learning any technology: the strongest explanations acknowledge where the tradeoffs are real, rather than treating the dominant approach as universally correct for every situation.

Practically, this is the actual starting line for the next 30 days — everything from here (EC2, VPC, IAM, S3, RDS) builds directly on top of the account created today. Worth flagging for myself early: get in the habit of NOT using the root user day-to-day the moment IAM is covered, rather than treating that as an optional cleanup step later.

---

## 🔗 Resources
* [AWS Official Sign-Up Page](https://aws.amazon.com/)
* [AWS Free Tier](https://aws.amazon.com/free/)
* [AWS Root User Best Practices](https://docs.aws.amazon.com/accounts/latest/reference/root-user-best-practices.html)
* [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*