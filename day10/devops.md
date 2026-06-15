![Progress](https://img.shields.io/badge/Progress-10%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 10 — Deploying NodeJS on EC2 + Top 15 AWS Services for DevOps

## 📝 Topic: End-to-End Deployment Workflow + The AWS Services That Actually Matter

**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 13, 2026

---

## 🎯 Learning Objectives

- Deploy a NodeJS application end-to-end — from local machine to a public EC2 instance.
- Understand the role of `.env` files and why they're critical for credential management.
- Follow the IAM best practice of never using the root account for daily operations.
- Provision an EC2 instance, generate a key pair, and SSH into it securely.
- Install a full Node/npm/Git toolchain on a fresh Ubuntu server.
- Open inbound security group rules to expose an app to the internet.
- Know the Top 15 AWS services every DevOps engineer should be fluent in — and why.

---

## 💻 Part 1 — Local Setup & Testing

Before anything touches AWS, the application must work locally. This isolates "is my code broken" from "is my infrastructure broken" — two completely different problem spaces.

### 📥 Step 1: Clone the Repository

```bash
git clone https://github.com/username/nodejs-app.git
cd nodejs-app
```

### 🔐 Step 2: Configure Environment Variables

Sensitive values (API keys, database URLs, payment gateway secrets) never go in source code. They go in a **hidden `.env` file**.

```bash
# Create the .env file
touch .env
vim .env
```

```bash
# .env
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxx
PORT=3000
```

> ⚠️ **Critical:** Add `.env` to `.gitignore` immediately. If a Stripe secret key ends up in a public GitHub repo, it can be scraped by bots within minutes and used for fraud.

```bash
# .gitignore
.env
node_modules/
```

### 📦 Step 3: Install Dependencies

```bash
npm install
```

This reads `package.json` and downloads every dependency into `node_modules/`. This step is why `node_modules/` is gitignored — it's regenerated from `package.json` + `package-lock.json`, never committed.

### ✅ Step 4: Test Locally

```bash
npm run start
```

```
Server running on http://localhost:3000
```

Open `localhost:3000` in a browser. **Confirm the app works before touching AWS.** Debugging a broken app is 10x harder once you add SSH, security groups, and remote environments into the mix.

---

## 🛡️ Part 2 — AWS Preparation: IAM Before EC2

### 👤 Why an IAM User, Not Root

The AWS root account has **unlimited power** over the entire account — billing, security, every service, every region. It should never be used for daily operations.

```
Root account:
  ❌ Used for daily tasks
  ❌ Access key shared across team
  ❌ No granular permission control
  ✅ Used ONLY for account-level changes (closing account, billing setup)

IAM User:
  ✅ Used for daily operations
  ✅ Scoped permissions — only what's needed
  ✅ Can be revoked individually without affecting the account
  ✅ MFA enabled per user
```

**Creating an IAM user:**

```
AWS Console → IAM → Users → Create User
  Username: devops-engineer
  Permissions: Attach policies directly
    → AmazonEC2FullAccess (for this task)
    → Or scope further with a custom policy

→ Generate access keys for CLI use (if needed)
→ Enable MFA for console login
```

> **Why this matters in an interview:** "I always use IAM users with least-privilege policies, never root" is one of the most commonly checked security fundamentals for DevOps roles.

---

## ☁️ Part 3 — EC2 Provisioning

### 🖥️ Launch the Instance

```
AWS Console → EC2 → Launch Instance

Name:           nodejs-app-server
AMI:            Ubuntu Server 22.04 LTS  ← Free Tier eligible
Instance Type:  t2.micro                 ← Free Tier eligible

Key pair:       Create new key pair
  Name:    nodejs-app-key
  Type:    RSA
  Format:  .pem  (for SSH from Linux/macOS/MobaXterm)

⚠️ Download the .pem file NOW — AWS never shows it again

Network settings:
  ✅ Allow SSH (port 22) from My IP
  (Port 3000 will be opened later, after deployment)

Storage: 8 GB (default — sufficient)

→ Launch Instance
```

### 🔑 Set Correct Permissions on the Key

```bash
chmod 400 nodejs-app-key.pem
```

> If permissions are too open, SSH refuses to use the key with: `UNPROTECTED PRIVATE KEY FILE!`

---

## 🔌 Part 4 — Connecting & Configuring the Remote Server

### 🖧 SSH Into the Instance

```bash
ssh -i nodejs-app-key.pem ubuntu@<EC2_PUBLIC_IP>
```

```
ubuntu@ip-172-31-x-x:~$
```

### 🔄 Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

> Always run this first on a fresh instance. Package lists on a brand-new AMI can be hours or days old.

### 🛠️ Install NodeJS, npm, and Git

```bash
# Install Git
sudo apt install git -y

# Install Node.js (using NodeSource for a current LTS version)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs -y

# Verify installations
node -v    # v20.x.x
npm -v     # 10.x.x
git --version
```

> **Why NodeSource instead of `apt install nodejs` directly?** Ubuntu's default repos often carry outdated Node versions. NodeSource provides current LTS releases.

---

## 🚀 Part 5 — Deploying the Application

### 📥 Clone the Repository on the Server

```bash
git clone https://github.com/username/nodejs-app.git
cd nodejs-app
```

### 🔐 Recreate the `.env` File

The `.env` file was correctly gitignored — meaning it **does not exist** on the cloned repo on the server. It must be recreated manually.

```bash
touch .env
vim .env
```

```bash
# .env (on the server — same content as local)
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxxxxxxxxxxx
STRIPE_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxx
PORT=3000
```

> **Production reality check:** Manually copying `.env` works for learning, but in real environments, secrets are managed via **AWS Secrets Manager**, **Parameter Store**, or CI/CD secret injection — never copy-pasted by hand at scale.

### ▶️ Install & Run

```bash
npm install
npm run start
```

```
Server running on http://localhost:3000
```

At this point, the app runs — but only the EC2 instance itself can reach `localhost:3000`. The outside world cannot.

---

## 🌐 Part 6 — Exposing the App to the Internet

### 🔓 Edit Inbound Security Group Rules

```
AWS Console → EC2 → Instances → select instance
  → Security tab → Security Groups → Edit inbound rules

Add rule:
  Type:        Custom TCP
  Port range:  3000
  Source:      0.0.0.0/0   ← anywhere on the internet
  Description: NodeJS app access

→ Save rules
```

### ✅ Verify Public Access

```
http://<EC2_PUBLIC_IP>:3000
```

The app is now live and reachable from anywhere.

> ⚠️ **Security note:** `0.0.0.0/0` means literally anyone on the internet can hit port 3000. For learning, this is fine. In production, this traffic should go through a **load balancer** with HTTPS, and port 3000 should never be directly exposed — use a reverse proxy (Nginx) on port 80/443 instead.

---

## 🗺️ Part 7 — The Complete Deployment Flow (Visual)

```
LOCAL MACHINE
  git clone → .env → npm install → npm run start → verify localhost:3000
        ↓
AWS SETUP
  Create IAM user (not root) → Launch EC2 (Ubuntu, t2.micro) → Generate .pem key
        ↓
REMOTE SERVER (via SSH)
  ssh -i key.pem ubuntu@<IP>
  sudo apt update
  Install: git, nodejs, npm
  git clone (same repo)
  Recreate .env
  npm install && npm run start
        ↓
EXPOSE TO INTERNET
  Edit Security Group → allow port 3000 from 0.0.0.0/0
        ↓
http://<EC2_PUBLIC_IP>:3000  ← live, public, working
```

---

## ☁️ Part 8 — Top 15 AWS Services for DevOps Engineers

> _"AWS has 200+ services. A DevOps engineer needs to be fluent in the ~15 that drive automation and efficiency."_

### 🏗️ Core Infrastructure

| Service | What it does                               | Why DevOps cares                                                        |
| ------- | ------------------------------------------ | ----------------------------------------------------------------------- |
| **EC2** | Virtual machines in the cloud              | The fundamental compute unit — everything runs on or references EC2     |
| **VPC** | Virtual private network for your resources | Network security, subnets, routing, traffic isolation                   |
| **EBS** | Block storage attached to EC2              | Persistent disks — data survives instance termination                   |
| **S3**  | Object storage for files/data              | Encrypted by default, scalable to petabytes, backbone of most pipelines |

```
VPC (network boundary)
 ├── Public Subnet  → EC2 instances with public IPs
 ├── Private Subnet → databases, internal services
 └── Security Groups → firewall rules per resource
```

---

### 🔐 Security & Management

| Service        | What it does                   | Why DevOps cares                                                    |
| -------------- | ------------------------------ | ------------------------------------------------------------------- |
| **IAM**        | Identity and Access Management | Every permission in AWS flows through IAM — users, roles, policies  |
| **KMS**        | Key Management Service         | Encrypts secrets, manages encryption keys for S3, EBS, RDS, etc.    |
| **CloudTrail** | API activity audit log         | Every API call is logged — "who did what, when" for security audits |

```bash
# Example: CloudTrail answers questions like:
# "Who terminated this EC2 instance at 3am yesterday?"
# "Which IAM user modified this S3 bucket policy?"
```

---

### 📊 Monitoring & Automation

| Service        | What it does                           | Why DevOps cares                                                              |
| -------------- | -------------------------------------- | ----------------------------------------------------------------------------- |
| **CloudWatch** | Centralized monitoring & observability | Metrics, logs, alarms, dashboards across all AWS resources                    |
| **Lambda**     | Serverless event-driven compute        | Auto-remediation — e.g., auto-stop idle EC2 instances at midnight             |
| **AWS Config** | Configuration tracking & compliance    | Detects configuration drift — "this S3 bucket became public, who changed it?" |

**Real-world Lambda + CloudWatch pattern:**

```
CloudWatch Alarm: "CPU > 90% for 5 minutes"
        ↓
Triggers Lambda function
        ↓
Lambda: scale up, send Slack alert, or auto-remediate
```

---

### 🔄 CI/CD & Containers

| Service          | What it does                            | Why DevOps cares                                             |
| ---------------- | --------------------------------------- | ------------------------------------------------------------ |
| **CodePipeline** | Orchestrates the full CI/CD workflow    | Connects source → build → test → deploy stages               |
| **CodeBuild**    | Compiles code, runs tests               | The "build" stage — produces deployable artifacts            |
| **CodeDeploy**   | Automates deployments                   | The "deploy" stage — rolling updates, blue/green deployments |
| **EKS**          | Managed Kubernetes                      | Runs Kubernetes without managing the control plane yourself  |
| **ECS**          | AWS-proprietary container orchestration | Simpler than Kubernetes, tightly integrated with AWS         |

```
CI/CD Pipeline Flow:

Source (GitHub) → CodeBuild (build & test) → CodeDeploy (deploy) → EC2/ECS/EKS
       ↑___________________ CodePipeline orchestrates all of this ___________________↑
```

**EKS vs ECS:**

|             | EKS                                  | ECS                               |
| ----------- | ------------------------------------ | --------------------------------- |
| Based on    | Kubernetes (open standard)           | AWS proprietary                   |
| Portability | Multi-cloud skills transfer          | AWS-only                          |
| Complexity  | Higher — full K8s API                | Lower — simpler AWS-native model  |
| Best for    | Teams already using K8s, multi-cloud | AWS-only shops wanting simplicity |

---

### 💰 FinOps & Logging

| Service                                         | What it does                 | Why DevOps cares                                                 |
| ----------------------------------------------- | ---------------------------- | ---------------------------------------------------------------- |
| **Billing & Cost Management**                   | Tracks and forecasts spend   | Identifying cost anomalies, setting budgets and alerts           |
| **ELK Stack** (Elasticsearch, Logstash, Kibana) | Log aggregation and analysis | Centralizes logs across microservices into searchable dashboards |

```
Microservice 1 → logs ─┐
Microservice 2 → logs ─┼─→ Logstash → Elasticsearch → Kibana (dashboard)
Microservice 3 → logs ─┘

One place to search: "show me all 500 errors in the last hour across all services"
```

---

### 📋 The Full List at a Glance

```
Core Infrastructure:    EC2, VPC, EBS, S3
Security & Management:  IAM, KMS, CloudTrail
Monitoring & Automation: CloudWatch, Lambda, AWS Config
CI/CD & Containers:      CodePipeline, CodeBuild, CodeDeploy, EKS, ECS
FinOps & Logging:        Billing & Cost Management, ELK Stack
```

> **Note:** These 15 cover standard interviews and general DevOps roles. Specialized domains — Machine Learning (SageMaker), Data Engineering (Glue, Redshift), IoT — require additional services on top of this foundation.

---

## 📖 Key Terms

| Term                | What it means                                                                     |
| ------------------- | --------------------------------------------------------------------------------- |
| **`.env` file**     | A hidden file storing environment-specific secrets — never committed to git       |
| **`.gitignore`**    | A file listing paths Git should never track — `.env`, `node_modules/`             |
| **`npm install`**   | Downloads all dependencies listed in `package.json`                               |
| **`npm run start`** | Runs the start script defined in `package.json`                                   |
| **IAM User**        | A scoped-permission identity for daily AWS operations — never use root            |
| **Root Account**    | The AWS account owner — has unrestricted access, reserved for account-level tasks |
| **Key Pair (.pem)** | RSA key used for SSH authentication into EC2 instances                            |
| **Security Group**  | A virtual firewall controlling inbound/outbound traffic to AWS resources          |
| **Inbound Rule**    | A security group rule that allows traffic INTO a resource                         |
| **0.0.0.0/0**       | CIDR notation meaning "all IP addresses" — fully open to the internet             |
| **VPC**             | Virtual Private Cloud — an isolated network environment within AWS                |
| **EBS**             | Elastic Block Store — persistent disk storage attached to EC2                     |
| **S3**              | Simple Storage Service — object storage for files at any scale                    |
| **KMS**             | Key Management Service — manages encryption keys                                  |
| **CloudTrail**      | Logs every API call made within an AWS account                                    |
| **CloudWatch**      | AWS's monitoring service — metrics, logs, alarms, dashboards                      |
| **Lambda**          | Serverless compute — runs code in response to events without managing servers     |
| **AWS Config**      | Tracks resource configuration changes and compliance over time                    |
| **CodePipeline**    | Orchestrates CI/CD stages from source to deployment                               |
| **CodeBuild**       | Compiles source code and runs tests — the "build" stage                           |
| **CodeDeploy**      | Automates application deployment to EC2, ECS, or Lambda                           |
| **EKS**             | Elastic Kubernetes Service — managed Kubernetes on AWS                            |
| **ECS**             | Elastic Container Service — AWS-native container orchestration                    |
| **ELK Stack**       | Elasticsearch + Logstash + Kibana — log aggregation and visualization             |

---

## 📂 Summary of Tasks

- ✅ Cloned a NodeJS repo and ran it locally with `npm install` + `npm run start`.
- ✅ Configured a `.env` file and added it to `.gitignore` before any commit.
- ✅ Understood: Why IAM users — not root — should be used for daily operations.
- ✅ Provisioned an EC2 instance (Ubuntu, t2.micro) and generated a `.pem` key pair.
- ✅ Connected via SSH and configured the remote server — `apt update`, Node, npm, Git.
- ✅ Deployed the cloned repo to the server and recreated the `.env` file.
- ✅ Opened port 3000 in the security group to expose the app publicly.
- ✅ Learned: The Top 15 AWS services across infrastructure, security, monitoring, CI/CD, and FinOps.

---

## 💡 My Takeaway

The deployment walkthrough made something click that documentation never does: **the `.env` file has to be recreated manually on the server because `.gitignore` did its job correctly.** It's a small thing, but it's a perfect example of how "best practice" (don't commit secrets) creates a manual step (recreate them elsewhere) — and in real production, that manual step gets replaced by Secrets Manager or CI/CD secret injection. Seeing the "naive" version first makes the "proper" version make sense later.

On the AWS services list: what stood out is how the 15 services map directly onto the SDLC phases from earlier days. EC2/VPC/S3 = infrastructure. IAM/KMS/CloudTrail = security across every phase. CodePipeline/CodeBuild/CodeDeploy = the Build→Test→Deploy automation DevOps owns. CloudWatch/Lambda/Config = the monitoring loop that feeds back into planning. It's not a random list — it's the DevOps lifecycle mapped onto AWS's service catalog.

---

## 🔗 Resources

- [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
- [NodeSource — Node.js Binary Distributions](https://github.com/nodesource/distributions)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [ELK Stack Overview — Elastic](https://www.elastic.co/elastic-stack)
- [EKS vs ECS Comparison — AWS](https://aws.amazon.com/eks/eks-vs-ecs/)

---

_Follow my journey! Feel free to ⭐ this repository to stay updated._
