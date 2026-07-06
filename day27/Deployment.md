![Progress](https://img.shields.io/badge/Progress-27%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 27 — Deploying a Static Website to AWS EC2 with Apache HTTPD

## 📝 Topic: End-to-End Static Site Deployment — GitHub to EC2 to Live on the Public Internet
**Instructor:** [Varun Bansal](https://github.com/) 
(hosted by [Abhishek Veeramalla](https://github.com/iam-veeramalla))

**Date:** July 03, 2026

---

## 🎯 Learning Objectives
* Push a local project to a fresh GitHub repository from scratch.
* Launch and configure an AWS EC2 instance with the right AMI, instance type, and Security Group rules.
* Connect to an EC2 instance securely via SSH using a `.pem` key pair.
* Install and configure Apache HTTPD on RHEL to serve a static site.
* Understand why a running web server process is required even after files are in place.
* Verify a live deployment by accessing the instance's public IP over HTTP.

---

## 📦 Phase 1 — Setting Up the Repository

### Goal
Move local project files into a remote GitHub repository so they can later be pulled onto the EC2 instance.

### Process

```bash
# 1. Create an empty repository on GitHub named "AWS-Demo"
#    (created via the GitHub UI — no README/gitignore, to avoid merge conflicts on first push)

# 2. Initialize the local directory as a git repo
git init

# 3. Stage and commit the project files
git add .
git commit -m "Initial commit - static website files"

# 4. Link the local repo to the remote GitHub repository
git remote add origin https://github.com/<username>/AWS-Demo.git

# 5. Push to the remote
git push -u origin master
```

> **Why this order matters:** the repo needs to exist and be reachable *before* the EC2 instance is even created — Phase 3 depends on being able to `git clone` this exact repository onto the server.

---

## ☁️ Phase 2 — Launching the AWS EC2 Instance

### Region Selection

```
Region chosen: Asia Pacific (Mumbai)
Reason: Lower latency for users geographically closer to this region
```

> General rule: always pick the AWS region closest to your expected user base to minimize round-trip latency.

### Instance Configuration

| Setting | Choice | Reason |
|---|---|---|
| **AMI** | Red Hat Enterprise Linux (RHEL) | Enterprise-grade Linux distribution, uses `yum` package manager |
| **Instance Type** | `t2.micro` | Free-tier eligible — ideal for a demo/learning workload |
| **Key Pair** | Newly created `.pem` file, downloaded locally | Required for secure SSH access — AWS never lets you download it again after creation |
| **Networking** | Public IP auto-assignment enabled | Without this, the instance has no address reachable from the public internet |
| **Security Group** | Allow SSH (port 22) + HTTP (port 80) | Port 22 for management access, Port 80 for serving the website to browsers |

```
Security Group rules configured:
  Inbound: SSH   (TCP 22)  ← Source: My IP (for secure admin access)
  Inbound: HTTP  (TCP 80)  ← Source: 0.0.0.0/0 (public web access)
```

> **Important safety note:** the `.pem` file downloaded during key pair creation is the *only* copy — if it's lost, there's no way to recover SSH access to that instance through the same key. It also needs restricted local permissions (`chmod 400 key.pem`) before SSH will accept it.

---

## 🔌 Phase 3 — Accessing and Deploying the App

### Connecting via SSH

```bash
ssh -i AWS-Demo-key.pem ec2-user@<public-ip>
```

### System Setup

```bash
# Switch to root user to avoid permission issues during setup
sudo su -

# Update all system packages
yum update -y

# Install Git so the repository can be cloned onto the instance
yum install git -y
```

```bash
# Clone the repository pushed to GitHub in Phase 1
git clone https://github.com/<username>/AWS-Demo.git
```

### Web Server Configuration

```bash
# Install Apache HTTPD
yum install httpd -y
```

```bash
# Apache serves files from this default directory
# Copy the cloned site's files into it
cp -r AWS-Demo/* /var/www/html/

# -r (recursive) is required to correctly copy
# any asset subfolders (images, CSS, JS) along with top-level files
```

> **Why `/var/www/html`:** this is Apache's default `DocumentRoot` — anything placed here is automatically what gets served when someone requests the server's root URL.

---

## ✅ Phase 4 — Launch and Verification

### The Missing Step Before Anything Works

```bash
systemctl start httpd
```

```
Before this command:
  Files are correctly in place at /var/www/html
  → but the website is STILL inaccessible

  Why: having the files present doesn't mean anything is
  listening on port 80 and serving them. Apache has to be
  actively RUNNING as a service for requests to be handled at all.
```

> **The lesson that ties the whole deployment together:** copying files into the right directory is necessary but not sufficient — a web server process has to be running to actually respond to incoming HTTP requests on that port. This is an easy step to forget when everything else appears correctly configured.

### Verification

```bash
# Optional: ensure Apache starts automatically on future reboots
systemctl enable httpd
```

```
Open a browser → http://<ec2-public-ip>
→ Static website loads successfully, publicly accessible over HTTP
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **EC2 (Elastic Compute Cloud)** | AWS's virtual machine service used to host the server here |
| **AMI (Amazon Machine Image)** | The OS template an EC2 instance boots from (RHEL, in this case) |
| **`t2.micro`** | A free-tier-eligible EC2 instance type, suitable for small demo workloads |
| **Key Pair (`.pem`)** | A public/private key pair used for secure SSH authentication to an instance |
| **Security Group** | AWS's virtual firewall controlling inbound/outbound traffic to an instance |
| **Public IP** | The internet-reachable address assigned to the instance |
| **`ec2-user`** | The default SSH login user for many AWS Linux-based AMIs |
| **`yum`** | The package manager used on RHEL/CentOS-based systems |
| **Apache HTTPD** | A widely used open-source web server, installed here to serve the static site |
| **`/var/www/html`** | Apache's default `DocumentRoot` — files placed here are served to visitors |
| **`systemctl start httpd`** | Starts the Apache service so it actively listens for and serves HTTP requests |
| **`systemctl enable httpd`** | Configures Apache to auto-start on future instance reboots |

---

## 📂 Summary of Tasks
- ✅ Created: A new GitHub repository (`AWS-Demo`) and pushed local static site files to it.
- ✅ Launched: An EC2 instance in the Mumbai region using a RHEL AMI and `t2.micro` instance type.
- ✅ Configured: A key pair for SSH access and a Security Group allowing SSH (22) and HTTP (80).
- ✅ Connected: To the instance via SSH using the downloaded `.pem` key.
- ✅ Updated: The system and installed Git and Apache HTTPD via `yum`.
- ✅ Deployed: Cloned site files into `/var/www/html` using `cp -r`.
- ✅ Identified: That files alone don't serve traffic — Apache must be actively running.
- ✅ Started: The `httpd` service and verified the site loads via the instance's public IP.

---

## 💡 My Takeaway

The step that stuck with me most was the gap between "files are in the right place" and "the website actually works." It's a subtle but important reminder that a web server is a *running process*, not just a folder full of static assets — `systemctl start httpd` is the line that turns a correctly configured filesystem into an actually reachable website. Skipping it (or forgetting it exists) is exactly the kind of silent gap that looks like a networking or DNS problem until you check whether the service is even running.

Doing this end-to-end — GitHub push, EC2 provisioning, SSH access, package installation, file deployment, service start — was a good reminder that "deploying a website" is really a chain of several independent systems (version control, cloud infrastructure, OS package management, a web server) that all have to be correctly wired together. Each phase individually is simple; the value is in seeing the full chain work end-to-end without skipping a link.

---

## 📈 Next Up
**Day 28:** Automating this EC2 deployment process — moving from manual SSH steps toward a scripted or CI/CD-driven deployment pipeline.

---

## 🔗 Resources
* [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
* [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
* [AWS Free Tier](https://aws.amazon.com/free/)
* [GitHub Docs — Creating a Repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*