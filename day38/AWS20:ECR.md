![Progress](https://img.shields.io/badge/Progress-38%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 20 — AWS ECR (Elastic Container Registry)

## 📝 Topic: Private, IAM-Integrated Container Image Storage on AWS
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 17, 2026

---

## 🎯 Learning Objectives
* Understand what ECR is by breaking down what each word in its name actually means.
* Compare ECR to Docker Hub and understand when each is the better fit.
* Authenticate the Docker CLI against ECR using AWS CLI credentials.
* Scope IAM permissions correctly for pushing images.
* Build, tag, and push a container image to a private ECR repository.
* Understand ECR's role as a foundational piece for AWS container orchestration (ECS/EKS).

---

## 📦 Part 1 — What Is AWS ECR?

### Definition

```
ECR = a fully managed AWS service for storing and
      managing CONTAINER IMAGES
```

### Breaking Down the Name

```
Elastic   → highly scalable and available — store a large
            volume of images without worrying about
            underlying infrastructure (same "Elastic"
            meaning as in EC2 from Day 03)

Container → relates to packaging application code and
            dependencies (Docker images, specifically)

Registry  → a storage location for those images —
            conceptually the same role as Docker Hub,
            Quay.io, or Google Container Registry (GCR)
```

---

## ⚖️ Part 2 — ECR vs. Docker Hub

### Primary Focus

```
Docker Hub → commonly used for PUBLIC images,
             widely accessible

ECR        → fundamentally designed for PRIVATE,
             ORGANIZATION-LEVEL image storage
```

### IAM Integration

```
ECR integrates SEAMLESSLY with AWS IAM:
  → Manage team permissions using EXISTING IAM
    user accounts
  → No separate Docker Hub credential system to
    manage alongside AWS credentials

  → Direct extension of the least-privilege IAM
    pattern from Day 02 — the same identity/policy
    model governing EC2, S3, and Lambda access now
    also governs who can push/pull container images
```

### Cloud Ecosystem Fit

```
ECR is PREFERRED when working within an AWS environment:
  → Native compatibility with EKS and ECS
  → No cross-platform credential friction —
    everything authenticates through the same IAM layer
```

| Aspect | Docker Hub | ECR |
|---|---|---|
| **Typical use** | Public images, broad community sharing | Private, org-scoped images |
| **Access control** | Separate Docker Hub credential system | Native AWS IAM integration |
| **Best fit** | Cross-platform, public-facing projects | AWS-native environments (ECS/EKS) |

---

## 🛠️ Part 3 — Practical Demonstration: Working with ECR

### Step 1: Setup

```bash
# Confirm AWS CLI is installed and configured
aws --version
aws configure
# (Reusing the exact CLI setup from AWS Day 10)
```

### Step 2: Authentication

```bash
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com
```

```
What's happening here:
  → aws ecr get-login-password fetches a temporary,
    short-lived authentication token via the AWS CLI
  → That token is piped DIRECTLY into `docker login`
    as the password, via --password-stdin

  → No long-lived Docker credentials are ever stored —
    the token is short-lived and regenerated per session,
    the same "temporary credentials over long-lived ones"
    principle behind IAM Roles (Day 02, Day 17)
```

### Step 3: IAM Permissions

```
If not using the root account (which should ALWAYS
be the case — Day 02's core rule), the IAM user/role
needs specific ECR permissions:

  ecr:GetAuthorizationToken
  ecr:InitiateLayerUpload
  ecr:UploadLayerPart
  ecr:CompleteLayerUpload
  ecr:PutImage
```

> **The least-privilege pattern showing up yet again:** just like the Lambda cost-optimization function (Day 18) needed exactly four specific EC2 permissions and nothing broader, pushing to ECR needs exactly these specific `ecr:*` actions — not a blanket `ecr:*` wildcard or broader container-service access.

### Step 4: Image Handling

```bash
# Build the image locally
docker build -t flask-app:latest .

# Tag it with the ECR repository URI
docker tag flask-app:latest \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/flask-app:latest

# Push to ECR
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/flask-app:latest
```

```
Why the TAG step matters specifically:
  → Docker doesn't know an image belongs in ECR just
    because it was built — the image must be explicitly
    tagged with ECR's full repository URI before `docker push`
    will send it to the correct destination
```

---

## ✅ Part 4 — Key Takeaways

```
1. Always use PRIVATE repositories for enterprise applications
   → Public exposure of proprietary application images is
     a real security/IP risk, not just a style preference

2. Leverage IAM for secure, granular access control
   → Same least-privilege discipline from every prior
     IAM-adjacent session in this series

3. ECR is NOT a free service
   → Clean up unused repositories to avoid ongoing charges —
     the exact same "unused resources quietly cost money"
     theme from the Day 18 cost-optimization project,
     just applied to container image storage specifically

4. ECR is a CRITICAL component for CI/CD pipelines
   when using AWS container services like ECS or EKS
   → The natural next step after building images in
     CodeBuild (Day 14) — instead of pushing to Docker
     Hub, an AWS-native pipeline would push to ECR
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS ECR** | AWS's fully managed, private container image registry |
| **Registry** | A storage service for container images (Docker Hub, ECR, GCR, Quay.io) |
| **`aws ecr get-login-password`** | CLI command generating a short-lived authentication token for Docker login to ECR |
| **`docker login --password-stdin`** | Authenticates Docker CLI using a piped-in password/token, avoiding storing it in shell history |
| **Repository URI** | The full ECR path (`<account-id>.dkr.ecr.<region>.amazonaws.com/<repo-name>`) an image must be tagged with before pushing |
| **`ecr:PutImage`** | The specific IAM permission required to push an image manifest to ECR |
| **Vendor Lock-In** | The risk of dependency on a single provider's proprietary service, relevant when comparing ECR to portable alternatives |

---

## 📂 Summary of Tasks
- ✅ Understood: ECR's name breakdown — Elastic (scalable), Container (Docker images), Registry (storage location).
- ✅ Compared: ECR (private, IAM-integrated) vs. Docker Hub (public-oriented, separate credentials).
- ✅ Set up: AWS CLI authentication and used it to generate a short-lived Docker login token for ECR.
- ✅ Scoped: IAM permissions to the specific `ecr:*` actions required for pushing images.
- ✅ Built, tagged, and pushed: A Docker image to a private ECR repository.
- ✅ Reinforced: Private-by-default, least-privilege IAM, and cost cleanup as ECR best practices.
- ✅ Noted: ECR's role as the natural AWS-native image destination for CI/CD pipelines feeding ECS/EKS.

---

## 💡 My Takeaway

The `aws ecr get-login-password | docker login --password-stdin` pattern is a nice, concrete example of the "temporary credentials over long-lived ones" principle finally showing up at the Docker CLI level specifically — the token is short-lived and regenerated per session rather than being a permanent Docker Hub-style password sitting in a config file somewhere. It's the same underlying idea as IAM Roles from Day 02 and CodeBuild's temporary CI credentials from Day 14, just applied to registry authentication this time.

This session also quietly closes a loop from Day 14's CI pipeline project — that pipeline pushed images to Docker Hub, but now I have the AWS-native alternative clearly in view: swap the `docker login`/`docker push` targets in that same `buildspec.yaml` to point at ECR instead, and the entire pipeline becomes fully AWS-native end to end, no external Docker Hub dependency at all. Worth actually doing that swap on my own CI pipeline rather than just noting it as a hypothetical.

The "ECR is not free, clean up unused repos" reminder is the exact same discipline as the Day 18 stale-snapshot cleanup project — worth remembering that "unused container images sitting in a private registry" is just as real a cost-optimization target as EBS snapshots or unattached Elastic IPs, and could plausibly be added as another resource type for that same Lambda cleanup function to check.

---


## 🔗 Resources
* [Amazon ECR Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)
* [ECR Authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html)
* [ECR IAM Permissions Reference](https://docs.aws.amazon.com/AmazonECR/latest/userguide/security_iam_service-with-iam.html)
* [ECR Pricing](https://aws.amazon.com/ecr/pricing/)
* [Docker CLI Push/Pull Reference](https://docs.docker.com/engine/reference/commandline/push/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*