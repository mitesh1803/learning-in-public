![Progress](https://img.shields.io/badge/Progress-33%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 09 — S3 (Simple Storage Service)

## 📝 Topic: Object Storage Fundamentals, Bucket Management, Security & Static Website Hosting
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 11, 2026
---

## 🎯 Learning Objectives
* Understand what S3 is and the problem it solves for organizations.
* Understand S3's durability guarantee and what "11 nines" actually means.
* Create and configure an S3 bucket, including naming rules and region scoping.
* Understand objects, metadata, encryption options, and multipart uploads.
* Learn S3 Storage Classes and how they map to cost vs. access frequency.
* Understand Versioning, Lifecycle Management, Replication, and Event Notifications.
* Configure bucket policies and IAM-based access control for S3.
* Host a static website directly from an S3 bucket, including CORS considerations.

---

## 📦 Part 1 — What Is AWS S3?

### Definition

```
S3 = Simple Storage Service

A highly scalable, highly available, and secure
OBJECT STORAGE service.
```

### The Problem It Solves

```
Organizations struggle with storing:
  → Application logs
  → Database dumps/backups
  → Media files (images, videos)

Traditional approach: manage physical storage hardware
  → Same overhead problems covered on AWS Day 01
    (maintenance, capacity planning, physical space)

S3's approach: rent effectively unlimited cloud storage,
  pay only for what's used, no hardware to manage
```

### Reliability: 11 Nines of Durability

```
S3 durability: 99.999999999% (eleven 9s)

What this actually means:
  → The statistical probability of losing a single object
    is near zero over a 100-YEAR period

  → This is achieved through automatic replication of data
    across multiple physical facilities within a region
```

> **Why this number matters practically:** durability (will my data survive?) is a distinct guarantee from availability (can I access my data right now?) — S3's extreme durability figure specifically addresses data LOSS risk, not momentary access interruptions.

---

## 🗂️ Part 2 — S3 Buckets: Creating and Configuring

### What Is a Bucket?

```
A bucket = a container for storing objects (files) in S3

Think of it as a top-level folder holding your data —
except this "folder" has a name that must be
GLOBALLY UNIQUE across ALL of AWS, not just your account.
```

### Bucket Naming Rules

```
Bucket names must:
  → Be globally unique across every AWS account, worldwide
  → Follow DNS naming conventions
  → Be 3-63 characters long
  → Contain only lowercase letters, numbers, periods, and hyphens
```

### Region Scoping

```
S3 is described as a "global service" — but each individual
BUCKET is scoped to a SPECIFIC AWS region.

Why this matters:
  → Reduces latency for users/applications near that region
  → May affect compliance with data-residency regulations
    (some industries/countries require data to stay
     within specific geographic boundaries)
```

### Bucket-Level Permissions and Policies

```
Access control is managed via:
  → IAM Policies      → attached to users/roles, define what
                         THAT IDENTITY can do
  → Bucket Policies    → attached to the BUCKET ITSELF, define
                         who can access THAT BUCKET

Both can apply simultaneously — the effective permission
is the intersection/evaluation of both together.
```

---

## 📤 Part 3 — Uploading and Managing Objects

### Uploading Objects

```
Methods available:
  → AWS Management Console (manual, UI-based)
  → AWS CLI
  → AWS SDKs (programmatic, in application code)
  → Direct HTTP uploads

Every object gets a unique KEY (its "name") within
the bucket — this key is how the object is retrieved later.
```

### Object Metadata

```
Each object carries metadata describing it:
  → Content type
  → Cache control settings
  → Encryption settings
  → Custom metadata (arbitrary key-value pairs you define)
```

### Server-Side Encryption (SSE) Options

| SSE Type | Who manages the encryption key |
|---|---|
| **SSE-S3** | Amazon-managed keys |
| **SSE-KMS** | AWS Key Management Service (more control, auditability) |
| **SSE-C** | Customer-provided keys (you supply and manage the key yourself) |

### Multipart Uploads

```
For LARGE objects:
  → Split the object into multiple parts
  → Upload each part IN PARALLEL
  → S3 combines them into the complete object once all parts arrive

Benefits:
  → Faster overall upload (parallelism)
  → RESUMABLE — if one part fails, only that part
    needs to be re-uploaded, not the entire file
```

### S3 Batch Operations

```
For performing bulk operations across large numbers
of objects at once:
  → Copying objects
  → Tagging objects
  → Restoring archived data

Useful when an operation needs to apply to thousands/millions
of objects — doing this one-by-one via individual API calls
would be impractically slow and expensive.
```

---

## 🏗️ Part 4 — Advanced S3 Features

### S3 Storage Classes

```
Different classes trade off COST vs. ACCESS FREQUENCY/SPEED:

  S3 Standard              → frequently accessed data
  S3 Standard-IA           → infrequently accessed, still needs fast retrieval
  S3 Glacier / Glacier Deep Archive → long-term archival, slow/cheap retrieval

General principle:
  Less frequently accessed data → cheaper storage class →
  but slower/more expensive to RETRIEVE when you do need it
```

> **Cost optimization strategy:** match the storage class to actual access patterns. Storing rarely-accessed backup data in S3 Standard wastes money; storing frequently-accessed application assets in Glacier makes them painfully slow and expensive to serve.

### Lifecycle Management

```
Define RULES for automatically transitioning objects
between storage classes, or deleting them, based on age:

Example rule:
  "After 30 days  → move to S3 Standard-IA"
  "After 90 days  → move to Glacier"
  "After 365 days → delete"

→ Fully automated — no manual intervention needed
  to keep storage costs optimized over time
```

### S3 Replication

```
Cross-Region Replication (CRR):
  → Automatically, asynchronously replicates objects
    to a bucket in a DIFFERENT region
  → Use case: disaster recovery, compliance requirements

Same-Region Replication (SRR):
  → Replicates within the SAME region
  → Use case: data resilience, low-latency access
    for a separate team/application reading a copy
```

### S3 Event Notifications and Triggers

```
Configure ACTIONS to fire automatically when specific
events occur in a bucket (e.g., object created, object deleted):

  → Trigger an AWS Lambda function
  → Send a message to Amazon SQS
  → Invoke other services via Amazon SNS

Example: uploading an image to S3 automatically triggers
a Lambda function that generates a thumbnail
```

---

## 🔒 Part 5 — Security and Compliance

### Bucket Security Considerations

```
Ensure properly configured:
  → Bucket policies
  → Access control (ACLs, IAM)
  → Encryption settings

→ Regularly MONITOR and AUDIT access logs
  for unauthorized activity
```

### Encryption: At Rest and In Transit

```
At rest:  Server-side encryption (SSE-S3, SSE-KMS, SSE-C)
In transit: SSL/TLS during data transfer (HTTPS)
```

### Access Logging and Monitoring

```
Enable ACCESS LOGGING to capture detailed records
of every request made to the bucket.

→ Monitor these logs, configure ALERTS for suspicious
  activity or unauthorized access attempts
```

---

## 🌐 Part 6 — Practical: Bucket Policies & Static Website Hosting

### Bucket Policies Overriding Broad IAM Access

```
Key scenario demonstrated:
  → An IAM user has BROAD S3 access via their IAM policy
  → A BUCKET POLICY on a specific bucket explicitly DENIES
    or restricts access for that same user

  Result: the bucket policy's restriction still applies,
  even though the IAM policy alone would have allowed it.
```

> **Why this matters:** bucket policies are a critical additional control layer — they let you lock down a *specific* sensitive bucket even for users/roles that otherwise have broad S3 permissions elsewhere in the account. This is the same "combine multiple independent layers of defense" pattern seen with Security Groups + NACLs.

### Static Website Hosting on S3

```
Steps:
  1. Enable "Static website hosting" in bucket Properties
  2. Set the index document (e.g., index.html)
  3. Configure PUBLIC ACCESS (disable the default
     "Block all public access" setting)
  4. Add a BUCKET POLICY explicitly allowing public
     READ access (s3:GetObject) for everyone
```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-site-bucket/*"
    }
  ]
}
```

> Note the parallel to the earlier EC2 static site deployment (Day 27): the underlying goal — serve static HTML/CSS/JS publicly — is the same, but S3 achieves it with zero servers to manage, patch, or scale at all.

### CORS (Cross-Origin Resource Sharing)

```
Needed when: the hosted website makes API calls to a
DIFFERENT domain than the one it's served from

Without CORS configured:
  → Browser blocks the cross-origin request by default
    (a browser security mechanism, not an S3 limitation)

With CORS configured on the bucket:
  → Explicitly allow specific origins/methods/headers
    to make cross-domain requests successfully
```

---

## 🛠️ Part 7 — Management, Monitoring & Troubleshooting

### Management Tools

```
AWS Management Console  → manual, UI-based
AWS CLI                 → scriptable, command-line
AWS SDKs / APIs         → programmatic, embedded in application code
```

### Monitoring with CloudWatch

```
CloudWatch can:
  → Monitor S3 metrics (request counts, error rates, storage size)
  → Set up ALARMS for specific conditions
  → Collect and analyze logs for troubleshooting/performance
```

### Common Errors and Debugging

```
Common S3 errors:
  → Access Denied           → check IAM policy + bucket policy together
  → Bucket Not Found        → verify bucket name/region
  → Exceeded Bucket Quota   → check account-level service limits

Debugging tools:
  → AWS CloudTrail (API call history/audit trail)
  → S3 access logs (detailed per-request records)
```

### Recovering Deleted Objects

```
If Versioning was enabled:
  → Deleted objects aren't truly gone — a "delete marker"
    is added, but previous versions remain retrievable

If Cross-Region Replication was configured:
  → A copy may still exist in the replicated destination bucket
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **S3 Bucket** | A globally-uniquely-named container for storing objects in S3 |
| **Object** | An individual file stored in S3, identified by a unique key |
| **Durability (11 nines)** | The statistical guarantee against data loss — 99.999999999% |
| **Availability** | Whether data can be accessed right now — a distinct guarantee from durability |
| **SSE (Server-Side Encryption)** | Encrypting data at rest in S3; options are SSE-S3, SSE-KMS, SSE-C |
| **Multipart Upload** | Splitting a large object into parts uploaded in parallel, resumable on failure |
| **Storage Class** | A cost/performance tier for stored objects (Standard, Standard-IA, Glacier, etc.) |
| **Lifecycle Management** | Automated rules to transition or delete objects based on age |
| **Cross-Region Replication (CRR)** | Automatic async replication of objects to a bucket in a different region |
| **Same-Region Replication (SRR)** | Automatic async replication of objects within the same region |
| **S3 Event Notification** | Automated triggers (Lambda, SQS, SNS) firing on bucket events like object creation |
| **Bucket Policy** | A JSON document attached to a bucket defining access permissions, independent of IAM |
| **CORS** | Browser-enforced mechanism controlling whether cross-domain requests are permitted |
| **Versioning** | Keeping multiple versions of an object to protect against accidental overwrite/deletion |

---

## 📂 Summary of Tasks
- ✅ Understood: What S3 is and the storage-management problem it solves.
- ✅ Understood: S3's 11-nines durability guarantee and how it differs from availability.
- ✅ Learned: Bucket naming rules and why buckets are region-scoped despite S3 being a global service.
- ✅ Learned: Object uploads, metadata, and the three server-side encryption options.
- ✅ Understood: Multipart uploads for large files and S3 Batch Operations for bulk actions.
- ✅ Learned: S3 Storage Classes and how to map cost to actual access frequency.
- ✅ Understood: Lifecycle Management, Replication (CRR/SRR), and Event Notifications.
- ✅ Practiced: Writing a bucket policy and confirming it can restrict access even with broad IAM permissions.
- ✅ Configured: Static website hosting on S3, including public access and CORS considerations.
- ✅ Learned: CloudWatch monitoring, common S3 errors, and object recovery via Versioning/Replication.

---

## 💡 My Takeaway

The bucket-policy-overriding-broad-IAM-access demonstration was the most important security detail today — it's easy to assume that once a user has wide S3 permissions via IAM, that's the whole story. Seeing a bucket policy explicitly restrict access for that same user reinforces that S3 access control is genuinely a TWO-LAYER system (IAM + bucket policy), evaluated together — not a single source of truth. It's the exact same "combine independent layers of defense" pattern from Security Groups + NACLs, just showing up again at the storage layer instead of the network layer.

The durability-vs-availability distinction is a subtlety worth keeping precise — "11 nines" is specifically about data survival, not about whether a request succeeds right now. Conflating the two would be a mistake in an interview or in real architecture decisions (e.g., choosing Glacier for durability is fine, but Glacier's retrieval latency is a separate availability/performance tradeoff entirely).

Static website hosting on S3 was a nice callback to the AWS Day 27 EC2 + Apache deployment — same end goal (serve static files publicly), completely different mechanism. Worth remembering S3 as the simpler, zero-server default for anything genuinely static, and reserving EC2 + a real web server for cases that need actual server-side logic.

---

## 🔗 Resources
* [Amazon S3 Documentation](https://docs.aws.amazon.com/s3/index.html)
* [S3 Storage Classes](https://aws.amazon.com/s3/storage-classes/)
* [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
* [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
* [S3 Versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
* [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*