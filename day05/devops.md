
![Progress](https://img.shields.io/badge/Progress-5%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 06 & 07 — VM Provisioning on AWS/Azure + Connecting via SSH

## 📝 Topic: Manual & Automated VM Creation + SSH into EC2 (Windows Guide)
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 9, 2026

---

## 🎯 Learning Objectives
* Understand the difference between manual and automated VM provisioning.
* Know all major automation methods for cloud infrastructure — CLI, SDK, IaC.
* Decide when to use Terraform vs provider-specific tools like CDK or ARM.
* Launch an AWS EC2 instance manually from scratch.
* Connect to a Linux EC2 instance from Windows using MobaXterm via SSH.
* Understand what a `.pem` key file is and why it's required.

---

## ⚙️ Part 1 — Manual vs Automated Provisioning

### 🖱️ Manual Provisioning
Creating VMs by clicking through the AWS Console or Azure Portal.

| Aspect | Reality |
|---|---|
| **Good for** | Learning, one-off experiments |
| **Bad for** | Production, repeated deployments |
| **Risk** | Human error — wrong region, wrong size, missing config |
| **Speed** | Slow — minutes per VM, not scalable |

> **The DevOps principle:** If you do something more than once, automate it.

In a real organization, a DevOps engineer never manually clicks through the console to provision infrastructure. Every VM, network, and storage resource is defined as code.

### 🤖 Automated Provisioning — Why It Matters

```
Manual:     Click → Fill form → Click → Wait → Hope you didn't miss anything
Automated:  Run script → Done ✅ — same result every single time
```

**Benefits of automation:**
* **No human error** — the script doesn't mistype a region
* **Speed** — provision 100 VMs in the same time as 1
* **Repeatability** — dev, staging, and production are identical
* **Auditability** — the code is the record of what was built

---

## 🛠️ Part 2 — Automation Methods

### Method 1: AWS CLI & API

Interact directly with AWS services via the command line or HTTP calls.

```bash
# Create an EC2 instance via AWS CLI
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --count 1
```

**When to use:** Quick scripting, one-time automation, CI/CD pipeline steps.

---

### Method 2: Python (Boto3)

AWS's official Python SDK — wraps the API in Python functions.

```python
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')

response = ec2.run_instances(
    ImageId='ami-0c02fb55956c7d316',
    InstanceType='t2.micro',
    KeyName='my-key-pair',
    MinCount=1,
    MaxCount=1
)

instance_id = response['Instances'][0]['InstanceId']
print(f"Launched: {instance_id}")
```

**When to use:** Complex logic, dynamic infrastructure, applications that manage their own resources.

---

### Method 3: CloudFormation Templates (CFT)

AWS-native Infrastructure as Code — define resources in YAML or JSON.

```yaml
# cloudformation.yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: t2.micro
      KeyName: my-key-pair
```

```bash
# Deploy the stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://cloudformation.yaml
```

**When to use:** AWS-only environments, when you need stack management (create/update/delete together).

---

### Method 4: AWS CDK (Cloud Development Kit)

Define infrastructure using real programming languages (TypeScript, Python, Java).

```typescript
// infrastructure.ts
import * as ec2 from 'aws-cdk-lib/aws-ec2';

const instance = new ec2.Instance(this, 'MyInstance', {
  instanceType: ec2.InstanceType.of(
    ec2.InstanceClass.T2,
    ec2.InstanceSize.MICRO
  ),
  machineImage: ec2.MachineImage.latestAmazonLinux(),
  vpc: vpc,
});
```

**When to use:** Teams that prefer code over config files, complex infra with conditions and loops.

---

### Method 5: Terraform

Open-source Infrastructure as Code tool — works across all cloud providers.

```hcl
# main.tf
resource "aws_instance" "my_vm" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  key_name      = "my-key-pair"

  tags = {
    Name = "DevOps-Instance"
  }
}
```

```bash
terraform init
terraform plan   # preview changes
terraform apply  # provision
```

**When to use:** Multi-cloud or hybrid cloud environments.

---

### 🤔 Terraform vs Provider-Specific Tools

| Factor | Terraform | CDK / CFT / ARM |
|---|---|---|
| **Cloud support** | AWS + Azure + GCP + any provider | Single cloud only |
| **New features** | Slight lag — community must build providers | Immediate — direct from the cloud vendor |
| **Best for** | Hybrid cloud, multi-cloud teams | AWS-only or Azure-only stacks |
| **State management** | External state file required | Managed by the cloud provider |

> **Rule of thumb:** Use Terraform when you operate across multiple clouds. Use CDK/CFT/ARM when you're all-in on one provider and need day-one access to new features.

---

## ☁️ Part 3 — AWS EC2: Hands-On Creation (Step by Step)

### 🖥️ Launching an EC2 Instance Manually

**Step 1: Open the EC2 Console**
```
AWS Console → Services → EC2 → Launch Instance
```

**Step 2: Name your instance**
```
Name: devops-practice-01
```

**Step 3: Choose an AMI (Operating System)**
```
Ubuntu Server 22.04 LTS  ←  Free Tier eligible ✅
Architecture: 64-bit (x86)
```

**Step 4: Choose Instance Type**
```
t2.micro  ←  1 vCPU, 1 GB RAM, Free Tier eligible ✅
```

**Step 5: Create a Key Pair (Critical)**
```
Key pair name: devops-key
Key pair type: RSA
Private key format: .pem   ←  for Linux/macOS/MobaXterm
                   .ppk   ←  for PuTTY on Windows only
```

> ⚠️ **Download the `.pem` file immediately.** AWS will never show it again. If you lose it, you cannot connect to the instance and must terminate it.

**Step 6: Network Settings**
```
Allow SSH traffic from: My IP   ←  security best practice
Allow HTTP traffic: ✅ (if hosting a web app)
```

**Step 7: Configure Storage**
```
8 GB gp2 (default)  ←  sufficient for learning
```

**Step 8: Launch**
```
Review → Launch Instance → Wait ~60 seconds for "Running" state
```

**Key information to note after launch:**
```
Instance ID:  i-0abc123def456
Public IP:    54.210.x.x       ←  changes every restart (use Elastic IP to fix)
Private IP:   172.31.x.x       ←  stays the same
```

---

## 🔷 Part 4 — Azure VM Overview

Azure's portal mirrors AWS's structure but uses different terminology:

| AWS | Azure Equivalent |
|---|---|
| EC2 Instance | Virtual Machine |
| AMI | VM Image |
| Security Group | Network Security Group (NSG) |
| Key Pair (.pem) | SSH public key |
| IAM Role | Managed Identity |
| CloudFormation | ARM Templates / Bicep |
| CDK | Azure Bicep / Pulumi |

**Azure advantage for DevOps teams:** Native integration with GitHub and Azure DevOps pipelines — CI/CD is tightly coupled to the portal.

---

## 🔐 Part 5 — Connecting to EC2 from Windows via MobaXterm

### ❓ Why MobaXterm over PuTTY?

| Tool | Experience |
|---|---|
| **PuTTY** | Requires converting `.pem` → `.ppk` using PuTTYgen — extra steps, confusing for beginners |
| **MobaXterm** | Accepts `.pem` files directly — one step fewer, cleaner UI |

### 📥 Step 1: Download MobaXterm

```
Search: "MobaXterm download"
→ mobaxterm.mobatek.net
→ Download: Home Edition (Installer Edition)
→ Install normally
```

### 🖥️ Step 2: Launch EC2 Instance

```
AWS Console → EC2 → Launch Instance
OS: Ubuntu  (username will be: ubuntu)
Instance type: t2.micro
Key pair: create new → download devops-key.pem
Launch → wait for Running state → copy Public IP
```

### 🔑 Step 3: Configure SSH Session in MobaXterm

```
MobaXterm → Session (top-left) → SSH

Fill in:
  Remote host:  54.210.x.x         ← your EC2 Public IP
  Username:     ubuntu              ← default for Ubuntu AMIs
  Port:         22                  ← default SSH port

→ Advanced SSH Settings tab:
  ✅ Use private key
  Browse → select devops-key.pem

→ OK
```

> **Username reference by AMI:**

| AMI | Default SSH Username |
|---|---|
| Ubuntu | `ubuntu` |
| Amazon Linux 2 | `ec2-user` |
| CentOS | `centos` |
| Debian | `admin` |

### ✅ Step 4: Verify the Connection

Once connected, you see the Ubuntu terminal prompt:

```bash
ubuntu@ip-172-31-x-x:~$
```

Run a test command to confirm the instance is working:

```bash
sudo apt update
```

Expected output:
```
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
...
Fetched 23.4 MB in 4s
Reading package lists... Done
```

Connection confirmed ✅

### 🔒 How SSH Authentication Works

```
Your machine has:  devops-key.pem   (private key — never share)
EC2 instance has:  public key       (stored in ~/.ssh/authorized_keys)

SSH handshake:
  1. You present your private key
  2. EC2 verifies it matches the stored public key
  3. Access granted — no password needed
```

This is **asymmetric key authentication** — more secure than passwords.

---

## ⚡ Part 6 — Automation Methods Quick Reference

```
One VM, learning:          → AWS Console (manual)
Scripting & quick tasks:   → AWS CLI
Complex logic in Python:   → Boto3
AWS-only, config-driven:   → CloudFormation (CFT)
AWS-only, code-driven:     → AWS CDK
Multi-cloud / hybrid:      → Terraform
Azure environment:         → ARM Templates / Bicep
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **EC2** | Elastic Compute Cloud — AWS's VM service |
| **AMI** | Amazon Machine Image — the OS template used to launch an instance |
| **Instance Type** | The size of the VM — CPU, RAM, network performance (e.g. t2.micro) |
| **Key Pair** | An RSA public/private key pair used for SSH authentication |
| **.pem file** | The private key file — downloaded once at EC2 creation, never regenerated |
| **Public IP** | The externally accessible IP address of the EC2 instance |
| **Private IP** | The internal IP — only accessible within the same VPC |
| **Elastic IP** | A static public IP that persists across instance restarts |
| **Free Tier** | AWS's free usage tier — t2.micro + 750 hrs/month for 12 months |
| **SSH** | Secure Shell — the protocol for remote terminal access over port 22 |
| **MobaXterm** | A Windows SSH client that accepts `.pem` files directly |
| **Boto3** | AWS SDK for Python — programmatic access to all AWS services |
| **CloudFormation** | AWS-native IaC — defines infrastructure in YAML/JSON templates |
| **AWS CDK** | Cloud Development Kit — define AWS infra in TypeScript, Python, etc. |
| **Terraform** | Open-source IaC tool — works across AWS, Azure, GCP, and others |
| **Hybrid Cloud** | Using multiple cloud providers simultaneously |
| **IaC** | Infrastructure as Code — managing infrastructure through code files |
| **ARM Templates** | Azure's native IaC format (Azure Resource Manager) |

---

## 📂 Summary of Tasks
- [x] Understood: Why manual provisioning doesn't scale and automation is non-negotiable.
- [x] Learned: All 5 automation methods — CLI, Boto3, CFT, CDK, Terraform.
- [x] Understood: When to use Terraform vs provider-specific tools.
- [x] Practiced: Launching an EC2 instance step-by-step on AWS.
- [x] Understood: What a `.pem` file is and why losing it locks you out.
- [x] Practiced: Connecting to EC2 from Windows using MobaXterm over SSH.
- [x] Understood: How asymmetric key authentication works (public/private key pair).
- [x] Verified: Confirmed connection by running `sudo apt update` inside the instance.

---

## 💡 My Takeaway

The biggest mindset shift today: **manual = learning, automated = production**. The moment I start repeating an action in the console, that action should become a script or a Terraform file. This isn't about being clever — it's about eliminating the human from the critical path wherever possible. One mistyped region code in a production deployment can cost real money and time.

On SSH: understanding the public/private key model made the `.pem` file less mysterious. The EC2 instance holds the public key. My `.pem` file holds the private key. SSH's job is to verify they match without ever transmitting the private key across the network. That's why losing the `.pem` is permanent — there's no "forgot password" for asymmetric keys.

---

## 📈 Next Up
**Day 06:** Linux fundamentals deep dive — navigating the file system, permissions, processes, and the commands every DevOps engineer uses daily inside a VM.

---

## 🔗 Resources
* [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
* [MobaXterm Download](https://mobaxterm.mobatek.net/download.html)
* [AWS EC2 Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [AWS CLI Docs](https://docs.aws.amazon.com/cli/latest/reference/ec2/)
* [Boto3 EC2 Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html)
* [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
* [AWS CDK Docs](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
* [SSH Key Authentication Explained](https://www.ssh.com/academy/ssh/public-key-authentication)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
