![Progress](https://img.shields.io/badge/Progress-39%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 26 — Cloud Migration to AWS

## 📝 Topic: The Five-Stage Migration Lifecycle & the 7 R's Strategy Framework
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 19, 2026

---

## 🎯 Learning Objectives
* Understand cloud migration as a long-term project lifecycle, not a single strategy applied once.
* Learn the five stages of a migration project: Preparation, Planning, Migrate, Monitor, Optimize.
* Learn the "7 R's" migration strategies and when each is appropriate.
* Understand database migration best practices, including backup and rollback contingency.

---

## 🔄 Part 1 — Migration as a Lifecycle, Not a Single Strategy

```
Common misconception: "migration" = picking one of the
                        7 R's and executing it once

Reality: a successful migration project follows FIVE
         DISTINCT PHASES, with the 7 R's being just
         the technical APPROACH chosen during ONE
         of those phases (Planning)
```

---

## 🏗️ Part 2 — The Five Stages of Cloud Migration

### Stage 1: Preparation

```
Assess the CURRENT architecture.

If using a MONOLITHIC application:
  → May need to TRANSFORM it into a MICROSERVICES
    architecture first
  → Ensures it's genuinely cloud-native and
    compatible with container orchestration
    (Kubernetes/ECS — direct callback to the
     orchestration concepts from Days 21/EKS track)
```

> **Why preparation comes before planning:** you can't meaningfully choose a migration STRATEGY (Rehost vs. Refactor, etc.) until you understand what you're actually starting with. A monolith heading toward Kubernetes has fundamentally different migration needs than an already-containerized microservice.

### Stage 2: Planning

```
Organize the migration into PHASES.

NOT all applications move at once:
  → PRIORITIZE by criticality
  → Start with LOW-RISK applications (proof of concept)
  → GRADUALLY migrate more mission-critical services
    as confidence and process maturity increase
```

> This mirrors a sound general engineering principle seen elsewhere in this series — the Day 18 Lambda cost-optimization project's "test manually before enabling automated scheduling" caution, just applied here at the scale of an entire migration program rather than a single function.

### Stage 3: Migrate

```
The EXECUTION phase.

Involves writing SCRIPTS to automate:
  → Resource provisioning (e.g., EC2 instances)
    — direct application of Day 24's Terraform work
  → Application deployment onto AWS
```

### Stage 4: Monitor

```
AFTER migration, observe application PERFORMANCE:
  → Use tools like CloudWatch (Day 16)
  → COMPARE metrics against the PREVIOUS on-premises
    performance BASELINE
  → Confirms the migration didn't silently degrade
    something that worked fine before
```

> **Why comparing against a baseline matters specifically:** "it works" isn't sufficient confirmation for a migration — "it performs at least as well as it did on-premises" is the actual bar. Without an explicit before/after comparison, subtle regressions (increased latency, resource contention) could go unnoticed until they become user-facing problems.

### Stage 5: Optimize

```
Evaluate RESULTS:
  → Cost savings achieved
  → Efficiency gains

REFINE the infrastructure further:
  → Improve performance
  → Reduce overhead
    (direct continuation of Day 18's cost-optimization
     mindset — Optimize isn't a one-time final step,
     it's an ongoing discipline)
```

```
Full lifecycle:
  Preparation → Planning → Migrate → Monitor → Optimize
       ↑___________________________________________|
              (Optimize feeds back into ongoing
               refinement, not a hard stop)
```

---

## 🔤 Part 3 — The 7 R's: Cloud Migration Strategies

```
These are the TECHNICAL APPROACHES chosen specifically
during the Planning stage — not the whole migration
process itself.
```

| Strategy | Description | When to Use |
|---|---|---|
| **Rehost** ("Lift and Shift") | Move applications with minimal changes | Rapid migration, low effort priority |
| **Replatform** ("Lift, Tinker, and Shift") | Move with small optimizations to leverage AWS features | Want some AWS-native benefits without a full rebuild |
| **Refactor / Re-architect** | Completely redesign to be cloud-native (e.g., monolith → microservices) | Long-term scalability/agility is the priority |
| **Relocate** | Shift infrastructure platforms entirely (e.g., to EKS or Red Hat OpenShift on AWS) | Moving to a managed platform without full re-architecture |
| **Retain** | Keep certain apps on-premises | Sensitive/legacy apps unsuitable for cloud |
| **Retire** | Decommission unused applications | App is no longer needed at all |
| **Repurchase** | Move to a different software product/SaaS | Existing app can be replaced by a cloud-native SaaS equivalent |

> **The realistic picture:** a single migration project almost never uses just ONE of these 7 R's uniformly. A large organization typically applies Rehost to some legacy apps, Refactor to others being modernized, Retire to genuinely unused systems, and Retain for anything with hard compliance/sensitivity constraints — all within the same overall migration program.

---

## 🗄️ Part 4 — Database Migration Best Practices

### Right Fit

```
Utilize MANAGED services like AWS RDS
  → Reduces ongoing maintenance overhead
  → Direct extension of the "let AWS manage the
    undifferentiated heavy lifting" theme from
    EC2 (Day 03) and Lambda (Day 17) — applied
    here specifically to database administration
```

### Backup Strategy

```
ALWAYS maintain backups during migration.

Contingency plan required:
  → If the CLOUD-based database encounters issues
  → Have a plan to ROUTE TRAFFIC BACK to the
    ON-PREMISES database (in staging/pre-production)

  → This is a genuine rollback safety net — migration
    isn't a one-way door until confidence is established
```

> **Why this matters more for databases than almost any other resource:** losing application server capacity is inconvenient; losing or corrupting production DATA during a migration is often irreversible. The explicit "route traffic back to on-prem" contingency reflects that asymmetry in risk.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Preparation Stage** | Assessing current architecture, potentially transforming monoliths to microservices first |
| **Planning Stage** | Organizing migration into prioritized phases, choosing a 7 R's strategy |
| **Migrate Stage** | The execution phase — scripting resource provisioning and deployment |
| **Monitor Stage** | Post-migration performance observation against an on-premises baseline |
| **Optimize Stage** | Ongoing refinement for cost and performance after migration |
| **Rehost (Lift and Shift)** | Moving an app with minimal changes |
| **Replatform** | Moving an app with small AWS-leveraging optimizations |
| **Refactor / Re-architect** | Fully redesigning an app to be cloud-native |
| **Relocate** | Shifting to a different managed platform (e.g., EKS, ROSA) |
| **Retain** | Deliberately keeping an app on-premises |
| **Retire** | Decommissioning an unused app |
| **Repurchase** | Replacing an app with a SaaS equivalent |
| **AWS RDS** | AWS's managed relational database service, reducing DB administration overhead |

---

## 📂 Summary of Tasks
- ✅ Understood: Migration as a five-stage lifecycle, not a single one-time strategy choice.
- ✅ Learned: Preparation — assessing architecture and potentially refactoring monoliths first.
- ✅ Learned: Planning — phased, criticality-prioritized migration starting with low-risk apps.
- ✅ Learned: Migrate — the scripted execution phase, directly connecting to Day 24's Terraform work.
- ✅ Learned: Monitor — comparing post-migration CloudWatch metrics against an on-prem baseline.
- ✅ Learned: Optimize — ongoing cost/performance refinement after migration.
- ✅ Learned: All 7 R's strategies and realistic scenarios for each.
- ✅ Understood: Database migration best practices — managed services (RDS) and mandatory backup/rollback contingency.

---

## 💡 My Takeaway

Framing the 7 R's as just ONE input into a single stage (Planning) within a larger five-stage lifecycle was the most useful reframe today — I'd previously thought of the 7 R's AS the migration strategy, full stop. Understanding that a real migration also requires an honest Preparation assessment beforehand and a genuine Monitor/Optimize commitment afterward makes the whole thing feel like an actual engineering discipline with a beginning, middle, and ongoing tail — not a single decision made once at the start.

The database backup/rollback contingency detail is the one piece of advice from today I'd treat as completely non-negotiable in any real migration I'm ever involved in — the asymmetry between "an app server has a bad day" and "production data gets corrupted or lost" is significant enough that having an explicit, tested path back to the on-premises database isn't optional caution, it's a baseline requirement.

Seeing Terraform (Day 24), CloudWatch (Day 16), and managed services like RDS all show up again here as the actual TOOLS underpinning the Migrate and Monitor stages was a good confirmation that this series' individual topics were never really standalone — they're the building blocks this final strategic-thinking session assembles into an actual real-world project structure.

---

## 🔗 Resources
* [AWS Cloud Adoption Framework (CAF)](https://aws.amazon.com/cloud-adoption-framework/)
* [AWS Migration Strategies (7 R's)](https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html)
* [Amazon RDS Documentation](https://docs.aws.amazon.com/rds/index.html)
* [AWS Database Migration Service (DMS)](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html)
* [AWS Terraform Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*