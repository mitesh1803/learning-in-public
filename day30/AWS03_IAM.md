![Progress](https://img.shields.io/badge/Progress-30%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 03 — IAM (Identity and Access Management)

## 📝 Topic: Authentication, Authorization, Users, Policies, Groups & Roles
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 08, 2026

---

## 🎯 Learning Objectives
* Understand the distinction between Authentication ("who are you?") and Authorization ("what can you do?").
* Understand why defaulting every user to root access is dangerous.
* Learn IAM's four core components: Users, Policies, Groups, and Roles.
* Create an IAM user and observe what happens with zero attached policies.
* Grant permissions using AWS Managed Policies.
* Move users into Groups to scale permission management efficiently.
* Understand why Roles (not Users) are the correct mechanism for applications and services.

---

## 🔐 Part 1 — Core Concepts: Authentication & Authorization

### The Two Distinct Questions IAM Answers

```
Authentication: "Who are you?"
  → Verifying identity — e.g., logging into the AWS account
  → Happens FIRST, before anything else is evaluated

Authorization: "What are you allowed to do?"
  → Defining permitted actions for an ALREADY-authenticated identity
  → Happens AFTER authentication succeeds
```

### The Problem Without IAM

```
Without IAM, every user effectively defaults to "root" access:
  → Full, unrestricted control over the entire AWS account
  → Can delete databases, terminate instances, remove entire
    infrastructure — with no guardrails whatsoever

  A single mistake, a single compromised credential,
  and the blast radius is the ENTIRE account.
```

### The Solution: Least-Privilege Access

```
IAM lets administrators grant ONLY the permissions
a user or service genuinely needs — nothing more.

Principle of Least Privilege:
  → If a user only needs to read from S3,
    they get read-only S3 access — not admin access,
    not write access, not access to unrelated services
```

> **Why this matters:** least privilege isn't about distrust of individual users — it's about minimizing the damage any single compromised account, honest mistake, or bug in an automated process can cause.

---

## 🧩 Part 2 — Key Components of IAM

| Component | What it is | Example |
|---|---|---|
| **Users** | Individual identities for people needing AWS access | A specific engineer's login |
| **Policies** | JSON documents explicitly defining permissions | "Allow S3 read-only access" |
| **Groups** | Logical containers used to organize users for shared permissions | `Developers`, `QA` |
| **Roles** | Temporary security credentials, primarily for applications/services (not humans) | An EC2 instance needing DB access |

### Policies — The Actual Permission Definitions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

> A policy is just structured, explicit permission data — Allow/Deny on specific Actions against specific Resources. Nothing is granted implicitly; if it's not in a policy attached (directly or via a group/role) to an identity, that identity can't do it.

### Groups — Why They Exist

```
Without Groups:
  New developer joins → manually attach 5 different policies
  to their individual user, one at a time, every single time

With Groups:
  New developer joins → add them to the "Developers" group
  → they instantly inherit every policy already attached to that group
```

> **Automation benefit:** Groups turn a repetitive, error-prone manual task (attaching the same policies to every new hire) into a single, consistent action — add user to group, done.

### Roles — Why Applications Need a Different Mechanism Than Users

```
Users  → built for PEOPLE logging in with persistent credentials
Roles  → built for APPLICATIONS/SERVICES needing TEMPORARY,
         auto-rotating credentials

Example: An EC2 instance needs to read from a database
  → Attach a Role to the EC2 instance
  → The instance assumes that Role and gets temporary credentials
  → No long-lived access keys are hard-coded into the application
```

```
Roles are also used for:
  → Cross-account access
    (allowing a trusted identity in Account B to access
     specific resources in Account A, without creating
     a duplicate user in Account A)
```

> **Why not just create a "user" for the application instead?** Long-lived credentials (like a user's access keys) sitting inside application code or config files are a major security liability if leaked. Roles solve this by issuing short-lived, automatically rotating credentials instead.

---

## 🛠️ Part 3 — Practical Demonstration

### Step 1: Creating an IAM User

```
IAM Console → Users → Add User
  → Username: dev-user-1
  → Access type: AWS Management Console access
  → Auto-generated password
  → "User must create a new password at next sign-in" ✔ enabled
```

> Forcing a password reset on first login is a basic hygiene practice — it ensures the admin-generated temporary password never becomes the user's permanent credential.

### Step 2: Authorization Failure (No Policies Attached)

```bash
# Logged in as dev-user-1, with ZERO policies attached
aws s3 ls

# Result:
# An error occurred (AccessDenied) when calling the ListBuckets operation:
# Access Denied
```

> **This is the demonstration's key moment:** authentication succeeded (the login worked) but authorization failed completely — proving that having valid credentials and having permissions are two entirely separate things in IAM's model.

### Step 3: Granting Permissions via AWS Managed Policies

```
IAM Console → Users → dev-user-1 → Add Permissions
  → Attach existing policy directly
  → Select: AmazonS3FullAccess (an AWS Managed Policy)
```

```bash
# Re-running the same command after attaching the policy:
aws s3 ls
# 2026-06-01  my-app-bucket
# 2026-06-15  backup-bucket
```

> **AWS Managed Policies** are pre-built, AWS-maintained policy documents covering common permission sets (like full S3 access) — using them avoids writing custom JSON policies from scratch for standard use cases.

### Step 4: Group Management for Scalable Permissions

```
IAM Console → User Groups → Create Group
  → Name: Development
  → Attach Policy: AmazonS3FullAccess (moved from the user level to the group level)

Then: Add dev-user-1 (and any future developers) to the Development group
```

```
Before (policy on individual user):
  dev-user-1 → AmazonS3FullAccess (attached directly)

After (policy on group):
  Development group → AmazonS3FullAccess (attached once)
    ├── dev-user-1
    ├── dev-user-2   (future hire — just added to group, no policy work needed)
    └── dev-user-3
```

---

## ✅ Part 4 — Best Practices

```
1. Avoid Root
   → Never use the root account for daily operational tasks
   → Create a dedicated IAM user (with appropriate permissions)
     immediately after account setup, and use that instead

2. Automation via Groups
   → Attach policies to Groups, not individual Users, whenever
     multiple people need the same access
   → Eliminates repetitive manual policy attachment for every new hire

3. Roles for Non-Human Access
   → Any application, service, or EC2 instance needing AWS access
     should use a Role, never a User's long-lived access keys
   → Ensures credentials are temporary and automatically rotated
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Authentication** | Verifying WHO an identity is (e.g., a successful login) |
| **Authorization** | Verifying WHAT an authenticated identity is permitted to do |
| **IAM** | AWS's Identity and Access Management service — governs both authentication and authorization |
| **Least-Privilege Access** | Granting only the minimum permissions necessary, nothing more |
| **IAM User** | An individual identity created for a specific person needing AWS access |
| **IAM Policy** | A JSON document explicitly defining Allow/Deny permissions on specific actions/resources |
| **IAM Group** | A container of users used to attach shared policies once, instead of per-user |
| **IAM Role** | Temporary, auto-rotating credentials used by applications/services or for cross-account access |
| **AWS Managed Policy** | A pre-built, AWS-maintained policy covering common permission sets (e.g. `AmazonS3FullAccess`) |
| **Cross-Account Access** | Using Roles to let a trusted identity in one AWS account access resources in another |
| **AccessDenied Error** | The error returned when an authenticated identity lacks the required permission for an action |

---

## 📂 Summary of Tasks
- ✅ Understood: The distinction between Authentication (identity) and Authorization (permissions).
- ✅ Understood: Why defaulting to root access for every user is a major security risk.
- ✅ Learned: IAM's four core components — Users, Policies, Groups, Roles.
- ✅ Created: A new IAM user with console access and a forced password reset on first login.
- ✅ Confirmed: A user with zero attached policies gets an `AccessDenied` error on basic actions (e.g. `aws s3 ls`).
- ✅ Granted: S3 access to the user via the `AmazonS3FullAccess` AWS Managed Policy.
- ✅ Created: A `Development` IAM Group and moved the policy attachment from user-level to group-level.
- ✅ Reinforced: Avoid root for daily work, use Groups for scaling permissions, use Roles for non-human access.

---

## 💡 My Takeaway

The authorization-failure demo was the most useful part of today's session — actually seeing `aws s3 ls` return `AccessDenied` for a user with a perfectly valid login makes the Authentication-vs-Authorization split concrete instead of abstract. It's the same "authenticated but not authorized" distinction I already internalized from the Kubernetes RBAC session (Day 27 of the K8s track) — good to see the exact same security model showing up consistently across both Kubernetes and AWS. That's clearly not a coincidence; it's the standard pattern for any multi-tenant system with real security requirements.

The Users → Groups shift (moving `AmazonS3FullAccess` from an individual user to a `Development` group) is the kind of small operational change that pays off entirely in the future — the value isn't visible with one user, it's visible the moment a second, third, and tenth developer joins and needs the exact same access with zero extra policy work. Worth treating Groups as the default starting point rather than something to "refactor into" later.

The Roles section also directly connects to something I'll need soon — deploying EC2 instances that need to talk to other AWS services (S3, RDS, etc.) should use an instance Role from the start, not access keys baked into config files or environment variables.

---


## 🔗 Resources
* [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
* [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
* [AWS Managed Policies Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)
* [IAM Roles Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*