![Progress](https://img.shields.io/badge/Progress-50%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 08 — AWS Scenario-Based Interview Questions (VPC, EC2, IAM)

## 📝 Topic: 10 Common Interview Scenarios Recapping the First 7 Days of the Series
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 10, 2026

---

## 🎯 Learning Objectives
* Reframe the first 7 days of VPC/EC2/IAM concepts as answers to real interview scenarios.
* Practice articulating WHY a specific AWS component is the correct choice for a given constraint.
* Reinforce the Security Groups vs. NACLs distinction in scenario form.
* Review IAM's four core components in a quick-recall format.
* Understand VPC Peering and VPC Endpoints as two concepts not yet covered in depth elsewhere.

---

## 🌐 Part 1 — VPC Architecture & Networking Scenarios

### Scenario: High Availability & Scalability for a 2-Tier App

```
Q: How would you architect a highly available, scalable
   2-tier application on AWS?

A: Public Subnet  → Load Balancer
   Private Subnet → Application servers

   → Distribute BOTH subnets across MULTIPLE Availability Zones
   → Use an Auto Scaling Group to handle traffic spikes
     and replace unhealthy instances automatically
```

> This is a direct recap of the Day 32.1 project architecture — this scenario question is essentially "describe what you just built."

### Scenario: Restricting Outbound Internet Access

```
Q: How do you prevent instances in a specific subnet from
   reaching the internet at all?

A: Modify that subnet's ROUTE TABLE
   → Remove the route pointing to the Internet Gateway

   (Not a Security Group or NACL change —
    this is purely a routing-layer decision)
```

### Scenario: Internet Access for Private Subnet Instances

```
Q: Private instances need to download OS updates —
   how do you allow that without exposing them publicly?

A: NAT Gateway
   → Sits in a public subnet
   → Private instances route outbound traffic through it
   → NAT translates/masks the private IP in the process
```

### Scenario: Private Communication Between Instances

```
Q: How do instances communicate within the same VPC?
   What about across different VPCs?

A: Same VPC        → direct communication via private IP addresses
   Different VPCs   → VPC Peering
                      (creates a direct network connection
                       between two separate VPCs)
```

> **VPC Peering** is new terminology not covered in the earlier VPC deep dive (Day 04) — worth flagging as a gap to explore further: how peering connections are established, whether transitive peering is supported (it typically is NOT — each peering relationship is point-to-point).

### Scenario: Strict Network Access Control

```
Q: You need fine-grained, subnet-wide security control —
   what do you use?

A: Network ACLs (NACLs)
   → Subnet-level (vs. Security Groups, which are instance-level)
   → Acts as an EXTRA layer of defense for the entire subnet,
     independent of whatever individual instance Security
     Groups are configured
```

### Scenario: Fully Isolated Environments

```
Q: How do you create a highly isolated environment
   for sensitive workloads (e.g., compliance-restricted data)?

A: Private subnet WITH:
   → NO Internet Gateway route
   → NO Bastion Host access

   → Effectively unreachable from the internet in any direction,
     and inaccessible even for administrative SSH
     unless deliberately reconfigured
```

### Scenario: Accessing AWS Services Privately

```
Q: How do resources inside a VPC talk to services like
   S3 or DynamoDB WITHOUT going over the public internet?

A: VPC Endpoints
   → Allows private, in-VPC communication with AWS services
   → Traffic never traverses the public internet at all
```

> **VPC Endpoints** is the second new concept introduced in this recap session — distinct from VPC Peering (which connects two VPCs) — Endpoints connect a VPC directly to an AWS *service* without needing an Internet Gateway or NAT Gateway at all.

### Scenario: NACLs vs. Security Groups (Recap)

| Aspect | Security Groups | NACLs |
|---|---|---|
| **Level** | Instance | Subnet |
| **State** | Stateful (return traffic auto-allowed) | Stateless (return traffic needs explicit rule) |

> Direct recap of the AWS Day 05 deep dive — worth having this table memorized cold for interviews, since it's flagged here as one of the most commonly asked AWS networking questions.

---

## 🔑 Part 2 — IAM Scenarios

### IAM Components (Quick Recall)

```
Users     → Unique identities for people in the organization
Policies  → JSON documents defining authorization/permissions
Groups    → Collections of users, for simplified permission management
             (e.g., attach a policy to "Developers" group instead of
              individually to 500 separate users)
Roles     → Temporary permissions for AWS SERVICES
             (e.g., an EC2 instance needing S3 access)
```

> This is a direct recap of AWS Day 02 — the "Groups scale permission management, Roles are for non-human access" framing shows up again here, reinforcing it as one of the single most interview-relevant IAM concepts to have ready.

---

## 🚪 Part 3 — Administrative Access Scenario

### Bastion Host (Recap)

```
Q: How do you securely SSH/RDP into an instance that has
   NO public IP address at all?

A: Bastion Host ("jump server")
   → Deployed in a PUBLIC subnet
   → Admin connects to the Bastion first
   → Bastion Host is used to hop into the private instance

   Protocol depends on OS:
     Linux instances   → SSH
     Windows instances → RDP
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **VPC Peering** | A direct network connection between two separate VPCs, enabling private communication between them |
| **VPC Endpoint** | Allows private, in-VPC access to AWS services (S3, DynamoDB, etc.) without traversing the public internet |
| **Route Table** | Subnet-level rules directing where traffic is sent; removing the Internet Gateway route blocks outbound internet access entirely |
| **NAT Gateway** | Enables private-subnet outbound internet access while masking private IPs |
| **Security Group** | Instance-level, stateful firewall |
| **Network ACL (NACL)** | Subnet-level, stateless firewall, acting as an extra defense layer |
| **IAM User** | An individual identity for a person needing AWS access |
| **IAM Policy** | A JSON document defining specific permissions |
| **IAM Group** | A collection of users sharing the same attached policies |
| **IAM Role** | Temporary credentials, primarily used by AWS services rather than people |
| **Bastion Host** | A public-subnet jump server used for secure administrative access to private instances |
| **RDP** | Remote Desktop Protocol — used for administrative access to Windows instances (the Windows equivalent of SSH) |

---

## 📂 Summary of Tasks
- ✅ Recapped: High-availability 2-tier architecture using public/private subnets, multi-AZ, and ASG.
- ✅ Recapped: Restricting outbound internet access via Route Table modification.
- ✅ Recapped: NAT Gateway for private-subnet outbound internet access.
- ✅ Learned: VPC Peering for private communication between separate VPCs.
- ✅ Recapped: NACLs for subnet-wide, fine-grained network access control.
- ✅ Learned: Fully isolated environments via private subnets with no IGW route and no Bastion access.
- ✅ Learned: VPC Endpoints for private access to AWS services like S3/DynamoDB.
- ✅ Recapped: Security Groups (instance-level, stateful) vs. NACLs (subnet-level, stateless).
- ✅ Recapped: IAM's four components — Users, Policies, Groups, Roles.
- ✅ Recapped: Bastion Host as the secure administrative access pattern (SSH for Linux, RDP for Windows).

---

## 💡 My Takeaway

This session was less about new concepts and more about practicing the actual interview *framing* of things already covered — which turned out to be genuinely valuable on its own. Knowing what a NAT Gateway does is one skill; being able to answer "how would you allow private instances to get OS updates without exposing them publicly" fluently, in the shape of an interview answer, is a slightly different skill, and this recap format forces practice on the second one specifically.

VPC Peering and VPC Endpoints were the two genuinely new pieces of terminology here, and they're easy to conflate at a glance since both involve "private connectivity without the public internet." The distinction I want to keep clear: Peering connects two VPCs to each other, Endpoints connect one VPC to an AWS service. Worth digging into VPC Peering's non-transitive nature specifically before an actual interview, since that's a common follow-up gotcha question.

The isolated-environment scenario (private subnet, no IGW route, no Bastion access) is a nice concrete example of composing the individual pieces (subnets, routing, Bastion) into a specific security posture on demand — a good reminder that these aren't just individual features, they're building blocks meant to be combined deliberately based on the actual sensitivity of the workload.

---

## 📈 Next Up
**Day 33:** Extending the production architecture with HTTPS via ACM certificates on the ALB, and adding CloudWatch alarms tied to Auto Scaling policies.

---

## 🔗 Resources
* [VPC Peering Documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
* [VPC Endpoints Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html)
* [AWS Route Tables Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
* [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
* [Bastion Hosts on AWS](https://docs.aws.amazon.com/whitepapers/latest/aws-vpc-connectivity-options/bastion-hosts.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*