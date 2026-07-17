![Progress](https://img.shields.io/badge/Progress-37%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 19 — AWS CloudFront (CDN)

## 📝 Topic: Content Delivery Networks, Edge Locations & Serving a Private S3 Bucket Globally
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla) with guest [Piyush Sachdeva](https://github.com/)

**Date:** July 16, 2026
---

## 🎯 Learning Objectives
* Understand what a CDN is and the latency problem it solves.
* Understand how CloudFront uses Edge Locations to serve content close to end users.
* Learn CloudFront's key benefits: security, latency reduction, and cost optimization.
* Build a working CloudFront distribution in front of an S3 bucket.
* Understand Origin Access Identity (OAI) and why it keeps the S3 bucket private.
* Remember to tear down paid resources after hands-on practice.

---

## 🌍 Part 1 — What Is a CDN and Why Use CloudFront?

### Definition

```
CDN (Content Delivery Network) = a geographically
distributed network of servers that CACHE content
close to end-users, specifically to reduce latency
```

### The Problem Without a CDN

```
Central server (e.g., in one AWS region)
        │
        │  ← user is geographically FAR away
        ▼
  Multiple network "hops" required to reach the user

Result:
  → High latency
  → Buffering (for video/media content)
  → Generally poor user experience,
    purely due to physical distance and hop count
```

### The CloudFront Solution

```
CloudFront creates LOCAL COPIES of your content
at EDGE LOCATIONS distributed around the world.

User requests content
        ↓
Served from the NEAREST edge location
        ↓
(NOT from the far-away central storage/origin)
```

```
                    Central Origin
                    (S3 bucket, one region)
                          │
          ┌───────────────┼───────────────┐
          │                │                │
    Edge Location      Edge Location    Edge Location
    (North America)     (Europe)          (Asia)
          │                │                │
       Nearby            Nearby           Nearby
       Users             Users            Users
```

> **Why this dramatically reduces latency:** instead of every user's request traveling all the way to one central location, only the FIRST request per edge location (or a periodic refresh) needs to reach the origin — every subsequent nearby user gets served from a cached copy that's physically close to them.

### Key Benefits

```
Security:
  → S3 buckets don't need to be exposed directly to
    the internet — CloudFront sits in front of them

Latency:
  → Content served from the nearest edge location,
    not a single distant origin

Cost Optimization:
  → For high-traffic applications, serving cached
    content from edge locations reduces repeated
    load (and cost) on the origin
```

---

## 🛠️ Part 2 — Hands-On: Integrating S3 with CloudFront

### Step 1: S3 Bucket Setup

```
Create an S3 bucket to host static assets.

CRUCIAL settings for this specific setup:
  → Disable "Block all public access"
  → Enable static website hosting

(Note: this initial public-access setting gets
 effectively REVERSED later in Step 3 once OAI
 is configured — the bucket ends up NOT actually
 needing to be public once CloudFront is properly
 wired in front of it via OAI)
```

### Step 2: Upload Content

```
Upload static files:
  → index.html
  → CSS files, etc.

(Same static-site content pattern as the
 Day 09 S3 static website hosting demo)
```

### Step 3: Create CloudFront Distribution

```
CloudFront Console → Create Distribution

  Origin Domain: → select the S3 bucket created above
```

#### Origin Access Identity (OAI)

```
OAI = a special CloudFront identity used to access
      the S3 bucket ON BEHALF OF CloudFront ONLY

  → Ensures ONLY CloudFront can retrieve objects
    from the bucket — direct public access to the
    bucket itself is NOT required once this is set up
```

#### Updating the Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <OAI-ID>"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-site-bucket/*"
    }
  ]
}
```

```
Grants s3:GetObject permission SPECIFICALLY to the OAI
  → NOT to the public ("*") — a meaningfully different
    and more secure configuration than the Day 09
    static-hosting bucket policy, which allowed public
    "*" read access directly
```

> **This is the key security upgrade over the Day 09 approach:** instead of making the bucket publicly readable by anyone, only CloudFront (via its OAI) can read from it. The bucket effectively becomes private again — CloudFront is the sole, controlled gateway to its contents.

#### Geographic Scope

```
Choose which Edge Locations to use:
  e.g., "North America and Europe" (a common cost/coverage
  tradeoff — broader geographic coverage costs more)
```

### Step 4: Verification

```
Once the distribution finishes deploying:
  → Access the site via the CloudFront DISTRIBUTION
    DOMAIN NAME (e.g., d1234abcd.cloudfront.net)
  → Site loads successfully

  → Attempting to access the ORIGINAL S3 bucket URL
    directly now returns FORBIDDEN
    → Confirms the bucket is no longer directly
      publicly accessible — only reachable through
      CloudFront's OAI-gated path
```

---

## ⚠️ Part 3 — Important Reminders

### Cost Warning

```
CloudFront is explicitly flagged as a PAID service.

  → ALWAYS delete the distribution AND the S3 bucket
    after completing hands-on practice, to avoid
    unnecessary ongoing charges

  (Distributions can take some time to fully disable/
   delete — this isn't instantaneous like deleting
   most other resources)
```

### Security Recap

```
Using CloudFront in front of S3 allows you to:
  → Keep the S3 bucket PRIVATE
  → STILL serve its content globally, efficiently,
    and with low latency

  → This is a strictly BETTER security posture than
    the fully-public-bucket approach from Day 09 —
    worth defaulting to THIS pattern (CloudFront + OAI)
    for any real production static site, reserving the
    "public bucket policy" approach purely for quick
    learning demos.
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **CDN (Content Delivery Network)** | A geographically distributed network of caching servers reducing latency for end users |
| **AWS CloudFront** | AWS's managed CDN service |
| **Edge Location** | A CloudFront caching server location, physically distributed around the world |
| **Origin** | The source content location a CDN pulls from (an S3 bucket, in this project) |
| **Origin Access Identity (OAI)** | A special identity ensuring only CloudFront can access an S3 origin bucket |
| **Distribution** | A configured CloudFront setup — origin, edge behavior, caching rules — tied to a domain name |
| **Distribution Domain Name** | The CloudFront-provided URL used to access content through the CDN |
| **Bucket Policy (OAI-scoped)** | A policy granting `s3:GetObject` specifically to the OAI, rather than to the public |

---

## 📂 Summary of Tasks
- ✅ Understood: What a CDN is and the latency/hop-count problem it solves.
- ✅ Understood: How CloudFront's Edge Locations serve cached content near end users.
- ✅ Learned: CloudFront's three key benefits — security, latency reduction, cost optimization.
- ✅ Created: An S3 bucket configured for static website hosting.
- ✅ Created: A CloudFront Distribution with the S3 bucket as its Origin.
- ✅ Configured: Origin Access Identity (OAI) to restrict bucket access to CloudFront only.
- ✅ Updated: The bucket policy to grant `s3:GetObject` specifically to the OAI.
- ✅ Selected: A geographic scope (Edge Location coverage) for the distribution.
- ✅ Verified: The site loads via the CloudFront domain, while the raw S3 URL now returns Forbidden.
- ✅ Noted: The explicit cost warning — delete the distribution and bucket after practicing.

---

## 💡 My Takeaway

The direct comparison to Day 09's static hosting setup made this session land immediately — same underlying goal (serve static files publicly), but a genuinely more secure architecture. Day 09's approach required making the bucket policy allow `"Principal": "*"` — anyone, directly. Today's OAI approach scopes that same `s3:GetObject` permission to CloudFront specifically, and the bucket goes back to being effectively private. Seeing the S3 URL return Forbidden after the CloudFront distribution was live was a satisfying, concrete confirmation that the security model actually works as described, not just in theory.

This is a good one to file away as "the production-grade version" of static site hosting — worth defaulting to CloudFront + OAI + private bucket for anything real, and reserving the fully-public-bucket-policy approach purely for the fastest possible learning demo, exactly as flagged in today's session.

The cost warning is worth taking seriously given CloudFront distributions can take a while to actually finish deleting/disabling — unlike, say, deleting an S3 object, which is instantaneous. Worth building a habit of triggering the distribution deletion FIRST thing after finishing hands-on practice, rather than as an afterthought at the very end of a session, given the delay involved.

---
## 🔗 Resources
* [Amazon CloudFront Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)
* [CloudFront Origin Access Identity](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
* [CloudFront Edge Locations](https://aws.amazon.com/cloudfront/features/)
* [CloudFront Pricing](https://aws.amazon.com/cloudfront/pricing/)
* [S3 Static Website Hosting](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*