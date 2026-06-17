
![Progress](https://img.shields.io/badge/Progress-12%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 12 — Infrastructure as Code with Terraform

## 📝 Topic: IaC Concepts, Terraform Lifecycle, State Management & Remote Backends

**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 17, 2026

---

## 🎯 Learning Objectives

- Understand why multi-cloud infrastructure creates a learning overhead problem.
- Define Infrastructure as Code (IaC) and what problem it solves.
- Explain how Terraform works as an "API as Code" intermediary.
- Understand Terraform's limitations honestly — state file risks, GitOps friction, complexity.
- Install Terraform on Ubuntu, macOS, and CentOS.
- Write a `main.tf` to provision an AWS EC2 instance from scratch.
- Execute the full Terraform lifecycle: `init` → `plan` → `apply` → `destroy`.
- Understand the `terraform.tfstate` file — what it is and why it must never go in Git.
- Configure a remote backend with S3 + DynamoDB for team collaboration and state locking.
- Use modules for reusable, scalable infrastructure code.

---

## 🏗️ Part 1 — The Infrastructure Management Challenge

### 🛒 The Flipkart Scenario

Imagine you're a DevOps engineer at Flipkart. Today the stack runs on AWS. Six months later, the company decides to adopt a hybrid model — some workloads on Azure, some on-premises with OpenStack.

```
AWS infrastructure:
  → Uses CloudFormation Templates (CFT)
  → Learn CFT syntax, AWS-specific resources

Azure infrastructure:
  → Uses Azure Resource Manager (ARM) / Bicep
  → Completely different tool, different syntax

On-premises (OpenStack):
  → Uses Heat templates
  → Yet another tool, yet another language
```

Every time the platform changes, engineers must learn a **completely new toolchain** from scratch. This isn't a learning problem — it's an organizational cost problem. Knowledge doesn't transfer across tools.

---

## 💡 Part 2 — What is Infrastructure as Code (IaC)?

> _"IaC is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual configuration."_

### Before IaC vs After IaC

```
BEFORE (Manual / Console-driven):
  Engineer → clicks through AWS Console
           → makes mistakes, forgets steps
           → no record of what was built
           → cannot reproduce exactly
           → another engineer gets a different result

AFTER (IaC):
  Engineer → writes a config file
           → version controlled in Git
           → same file = same infrastructure, every time
           → reproducible across dev/staging/production
           → peer-reviewed like code
```

**The IaC contract:** If the code is the same, the infrastructure is the same. Always.

---

## 🌍 Part 3 — The Solution: Terraform

Developed by **HashiCorp**, Terraform is a single tool that works across every major cloud provider.

```
ONE TOOL:  Terraform (HCL — HashiCorp Configuration Language)

Manages:   AWS    → EC2, S3, RDS, VPC, IAM...
           Azure  → VMs, Storage, AKS, NSGs...
           GCP    → GCE, GCS, GKE...
           Others → Kubernetes, Datadog, GitHub, Cloudflare...
```

**The key advantage:**

```
Without Terraform:
  AWS project    → learn CFT
  Azure project  → learn ARM
  GCP project    → learn Deployment Manager

With Terraform:
  All projects   → learn HCL once
                   change the provider block, keep the rest
```

### 🔌 API as Code — How Terraform Actually Works

Every cloud provider exposes an API. Creating an EC2 instance is just an HTTP request to the AWS API.

```
Manual approach:
  Engineer → writes boto3 Python script → hits AWS API → creates EC2

Terraform approach:
  Engineer → writes HCL config
  Terraform → converts HCL → calls the appropriate provider API
           → AWS API / Azure API / GCP API
           → resource is created
```

Terraform acts as an **intermediary** — engineers describe _what_ they want, and Terraform figures out _which API calls_ to make to achieve it.

```
                    ┌─────────────────────┐
  main.tf (HCL)  →  │    Terraform Core   │  →  AWS API → EC2 instance
                    │  (provider plugins) │  →  Azure API → VM
                    └─────────────────────┘  →  GCP API → GCE instance
```

---

## ⚠️ Part 4 — Problems with Terraform (Honest Limitations)

> Understanding trade-offs is what separates senior from junior engineers.

| Problem                                    | What it means in practice                                                                                                                                   |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **State file is single source of truth**   | If the state file is lost, corrupted, or out of sync, Terraform loses track of what it manages — dangerous in production                                    |
| **Manual changes can't be auto-corrected** | If someone clicks in the AWS Console and changes a resource, Terraform doesn't know — its state is now wrong until the next plan detects drift              |
| **Not GitOps-friendly**                    | Terraform doesn't natively integrate with Flux or Argo CD — requires additional tooling (Atlantis, Terraform Cloud) to fit a GitOps workflow                |
| **Complexity at scale**                    | Large codebases with hundreds of resources become hard to maintain — module dependencies, state management, workspace management all add overhead           |
| **Scope creep**                            | Terraform tries to solve configuration management too (Ansible's job) — mixing IaC provisioning with configuration management in one tool creates confusion |

---

## 📦 Part 5 — Installation

### Ubuntu / Debian

```bash
# Add HashiCorp GPG key and repository
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

# Install
sudo apt update && sudo apt install terraform -y

# Verify
terraform --version
```

### macOS

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform --version
```

### CentOS / RHEL

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install terraform
terraform --version
```

---

## 📝 Part 6 — Writing Your First Project

### 📁 Project Structure

```
my-terraform-project/
├── main.tf          ← resource definitions
├── variables.tf     ← input variable declarations
├── outputs.tf       ← output values to display after apply
├── provider.tf      ← provider configuration
└── terraform.tfstate  ← auto-generated, DO NOT COMMIT TO GIT
```

### 🔧 `provider.tf` — Configure AWS

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  required_version = ">= 1.0"
}

provider "aws" {
  region = "us-east-1"
}
```

### 🖥️ `main.tf` — Create an EC2 Instance

```hcl
resource "aws_instance" "my_server" {
  ami           = "ami-0c02fb55956c7d316"   # Ubuntu 22.04 LTS us-east-1
  instance_type = "t2.micro"

  tags = {
    Name        = "terraform-demo-server"
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}
```

### 📤 `outputs.tf` — Useful Values After Creation

```hcl
output "instance_id" {
  description = "The ID of the created EC2 instance"
  value       = aws_instance.my_server.id
}

output "public_ip" {
  description = "The public IP address"
  value       = aws_instance.my_server.public_ip
}
```

---

## ⚙️ Part 7 — The Terraform Lifecycle

### The 4 Core Commands

```
terraform init    → download providers, set up backend
terraform plan    → preview what WILL change (dry run)
terraform apply   → make the changes (create/update/destroy resources)
terraform destroy → tear down all managed resources
```

---

### `terraform init`

```bash
terraform init
```

```
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "~> 5.0"...
- Installing hashicorp/aws v5.31.0...
- Installed hashicorp/aws v5.31.0 (signed by HashiCorp)

Terraform has been successfully initialized!
```

Downloads the AWS provider plugin and sets up the working directory. **Run this once per project or after any provider/backend changes.**

---

### `terraform plan`

```bash
terraform plan
```

```
Terraform will perform the following actions:

  # aws_instance.my_server will be created
  + resource "aws_instance" "my_server" {
      + ami                          = "ami-0c02fb55956c7d316"
      + instance_type                = "t2.micro"
      + tags                         = {
          + "Environment" = "dev"
          + "ManagedBy"   = "terraform"
          + "Name"        = "terraform-demo-server"
        }
      ... (many more auto-assigned attributes)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

**Reading the plan output:**

| Symbol     | Meaning                                  |
| ---------- | ---------------------------------------- |
| `+` green  | Resource will be CREATED                 |
| `~` yellow | Resource will be UPDATED in place        |
| `-/+` red  | Resource will be DESTROYED and recreated |
| `-` red    | Resource will be DESTROYED               |

> **Always review `terraform plan` before `terraform apply` in production.** This is the diff before the commit.

---

### `terraform apply`

```bash
terraform apply
```

Terraform shows the plan again and asks for confirmation:

```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.my_server: Creating...
aws_instance.my_server: Still creating... [10s elapsed]
aws_instance.my_server: Creation complete after 32s [id=i-0abc123def456]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:
  instance_id = "i-0abc123def456"
  public_ip   = "54.210.x.x"
```

For automated pipelines (no human approval):

```bash
terraform apply -auto-approve
```

---

### `terraform destroy`

```bash
terraform destroy
```

```
Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Enter a value: yes

aws_instance.my_server: Destroying...
aws_instance.my_server: Destruction complete after 28s

Destroy complete! Resources: 1 destroyed.
```

> **`terraform destroy` is permanent.** It terminates every resource Terraform manages in this workspace. In production, this requires careful access control — usually only CI/CD systems should have destroy permissions.

---

## 📋 Part 8 — The State File

### What is `terraform.tfstate`?

After `terraform apply`, Terraform writes a JSON file recording everything it created:

```json
{
  "version": 4,
  "terraform_version": "1.6.3",
  "resources": [
    {
      "type": "aws_instance",
      "name": "my_server",
      "instances": [
        {
          "attributes": {
            "id": "i-0abc123def456",
            "ami": "ami-0c02fb55956c7d316",
            "instance_type": "t2.micro",
            "public_ip": "54.210.x.x",
            ...
          }
        }
      ]
    }
  ]
}
```

This is the **single source of truth** — Terraform compares the current real-world infrastructure against this file on every `plan` to calculate what needs to change.

### ⚠️ Why It Must NEVER Go in Git

```
terraform.tfstate contains:
  → Database passwords
  → Private IP addresses
  → IAM access key IDs
  → Security group configurations
  → Any sensitive attribute of any resource

If committed to GitHub (even a private repo):
  → Secrets can be accessed by anyone with repo access
  → Leaked into git history — hard to fully remove
  → One accidental `git push` = security incident
```

```bash
# .gitignore — add these immediately
terraform.tfstate
terraform.tfstate.backup
.terraform/
*.tfvars           # may contain variable values including secrets
```

### 🔁 The State File Problem in Teams

```
Developer A: runs terraform apply → creates EC2 → state file on A's laptop
Developer B: runs terraform apply → Terraform doesn't know EC2 exists (state is on A's laptop)
             → tries to create another EC2 → conflict or duplicate resources
```

**State files on local machines = one of the most common Terraform anti-patterns.**

---

## ☁️ Part 9 — Remote Backend: S3 + DynamoDB

The production solution: store state remotely, lock it during operations.

### 📦 Step 1: Create S3 Bucket for State Storage

```hcl
# Create this BEFORE configuring the backend (chicken-and-egg problem)
resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-team-terraform-state-bucket"

  lifecycle {
    prevent_destroy = true   # prevent accidental deletion of state storage
  }
}

resource "aws_s3_bucket_versioning" "state_versioning" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"   # enables rollback to previous state versions
  }
}
```

### 🔒 Step 2: Create DynamoDB Table for State Locking

```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

**Why DynamoDB?** When Developer A runs `terraform apply`, DynamoDB creates a lock entry. If Developer B tries to run `terraform apply` simultaneously, they see:

```
Error: Error acquiring the state lock
  Lock Info:
    ID:        3d6f...
    Who:       developer-a@company.com
    Created:   2026-06-15 09:32:14
    Operation: OperationTypeApply
```

No corruption. No simultaneous conflicting applies. Developer B waits.

### ⚙️ Step 3: Configure the Backend

```hcl
# backend.tf (or in terraform {} block in main.tf)
terraform {
  backend "s3" {
    bucket         = "my-team-terraform-state-bucket"
    key            = "prod/ec2/terraform.tfstate"   # path within the bucket
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true   # encrypt state at rest
  }
}
```

```bash
# Re-initialize to migrate existing state to the remote backend
terraform init
```

```
Initializing the backend...
Do you want to copy existing state to the new backend? yes

Successfully configured the backend "s3"!
State is now stored remotely in S3.
```

### ✅ What Remote Backend Gives You

```
Before:
  State on Developer A's laptop → Developer B works blind → conflicts

After:
  State in S3 → everyone reads the same truth
  Lock in DynamoDB → only one apply at a time
  Versioning in S3 → roll back to previous state if something goes wrong
  Encryption → sensitive attributes protected at rest
```

---

## 🧩 Part 10 — Modules

### ❓ What is a Module?

A module is a **reusable package of Terraform configuration**. Instead of copy-pasting the same EC2 setup across 10 projects, write it once as a module.

```
modules/
└── ec2-instance/
    ├── main.tf       ← the EC2 resource definition
    ├── variables.tf  ← inputs (instance type, AMI, etc.)
    └── outputs.tf    ← outputs (instance ID, public IP)
```

### 📦 Using a Module

```hcl
# main.tf — calling the module
module "web_server" {
  source        = "./modules/ec2-instance"
  instance_type = "t2.micro"
  ami_id        = "ami-0c02fb55956c7d316"
  server_name   = "web-server-prod"
}

module "db_server" {
  source        = "./modules/ec2-instance"
  instance_type = "t3.medium"
  ami_id        = "ami-0c02fb55956c7d316"
  server_name   = "db-server-prod"
}
```

**Same module, different configurations** — no copy-paste, no drift between implementations.

---

## ⚠️ Part 11- Problems with Terraform

- State file is single source of truth.
- Manual changes to the cloud provider cannot be identified and auto-corrected.
- Not a GitOps friendly tool. Don't play well with Flux or Argo CD.
- Can become very complex and difficult to manage
- Trying to position as a configuration management tool as well.

---

## 🎤 Part 12 — Interview Questions

### Q1: Explain your Terraform setup

> "We use Terraform with an S3 remote backend for state storage and DynamoDB for state locking. All infrastructure code is version-controlled in Git with PR-based review. We use workspaces or separate state files per environment (dev/staging/prod). Changes go through `terraform plan` in CI before any `terraform apply` — humans review the plan output before approving."

### Q2: Why is state locking important?

> "Without locking, two engineers running `terraform apply` simultaneously could read the same state, make conflicting changes, and write corrupted state back to S3. DynamoDB locking ensures only one apply runs at a time — the second waits or fails with a clear error message."

### Q3: Where do you store the state file and why not in Git?

> "S3 with versioning enabled. Never in Git because the state file contains sensitive data — database passwords, private IPs, security configurations. Committing it to Git would expose those secrets to everyone with repo access and permanently embed them in git history."

### Q4: What is `terraform plan` and why is it critical?

> "`terraform plan` shows exactly what Terraform will create, modify, or destroy before any action is taken. It's the diff before the commit — you must review it before applying in production because some changes (like replacing an RDS instance) are destructive and irreversible."

---

## 📖 Key Terms

| Term                    | What it means                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------ |
| **IaC**                 | Infrastructure as Code — define and manage infrastructure through machine-readable files   |
| **HCL**                 | HashiCorp Configuration Language — Terraform's config syntax                               |
| **Provider**            | A plugin that lets Terraform interact with a specific platform (AWS, Azure, GCP, etc.)     |
| **Resource**            | A single infrastructure component defined in Terraform (EC2 instance, S3 bucket, VPC)      |
| **State File**          | `terraform.tfstate` — JSON file recording all resources Terraform manages                  |
| **Remote Backend**      | Storing state file in a shared location (S3) instead of locally                            |
| **State Locking**       | Preventing simultaneous applies using DynamoDB — avoids state corruption                   |
| **`terraform init`**    | Download providers, configure backend, set up working directory                            |
| **`terraform plan`**    | Dry run — preview what will change without making changes                                  |
| **`terraform apply`**   | Execute the planned changes against the real infrastructure                                |
| **`terraform destroy`** | Remove all resources managed by Terraform in this workspace                                |
| **Module**              | A reusable package of Terraform configuration — parameterised resource definitions         |
| **`+` in plan**         | Resource will be created                                                                   |
| **`~` in plan**         | Resource will be updated in place                                                          |
| **`-/+` in plan**       | Resource will be destroyed and recreated                                                   |
| **`prevent_destroy`**   | Lifecycle rule that prevents Terraform from accidentally destroying a resource             |
| **S3 Versioning**       | Keeps previous versions of the state file — enables rollback                               |
| **`-auto-approve`**     | Skips the manual confirmation prompt — used in CI/CD pipelines                             |
| **CFT**                 | AWS CloudFormation Templates — AWS-native IaC, replaced by Terraform in multi-cloud stacks |
| **Drift**               | When actual cloud resources differ from the Terraform state — caused by manual changes     |
| **Workspace**           | An isolated state file per environment (dev/staging/prod) within one codebase              |

---

## 📂 Summary of Tasks

- ✅ Understood: Why multi-cloud environments create a tool-learning overhead problem.
- ✅ Defined: Infrastructure as Code — machine-readable config over manual console clicks.
- ✅ Understood: How Terraform works as an API intermediary across cloud providers.
- ✅ Read and internalized: The 5 real limitations of Terraform (image above).
- ✅ Installed: Terraform on Ubuntu via the HashiCorp APT repository.
- ✅ Written: `provider.tf`, `main.tf`, and `outputs.tf` to provision an EC2 instance.
- ✅ Executed: The full lifecycle — `terraform init` → `plan` → `apply` → `destroy`.
- ✅ Understood: What the state file contains and why it must never be committed to Git.
- ✅ Configured: Remote backend — S3 for state storage, DynamoDB for state locking.
- ✅ Understood: Modules — reusable parameterised resource definitions.
- ✅ Practiced: 4 common Terraform interview questions with structured answers.

---

## 💡 My Takeaway

The state file is both Terraform's superpower and its biggest liability. It's the superpower because it lets Terraform calculate exactly what needs to change without querying every resource from scratch. It's the liability because it becomes a single point of failure — lose it, corrupt it, or let two people write to it simultaneously, and your infrastructure management breaks down. Every Terraform best practice — remote backend, state locking, versioning, encryption — exists to protect the state file.

The Problems with Terraform is the most honest framing I've seen for a tool that's often presented as the obvious choice. Drift detection is a real gap — Terraform won't auto-correct manual console changes until the next plan. For teams that live in GitOps workflows (Flux, Argo CD), that friction is real. Knowing these limitations isn't pessimism, it's what lets you make good architectural decisions about when Terraform is the right tool vs when something like Pulumi or AWS CDK might serve better.

---

## 🔗 Resources

- [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform Registry — AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices — HashiCorp](https://developer.hashicorp.com/terraform/language/style)
- [S3 Remote Backend Docs](https://developer.hashicorp.com/terraform/language/settings/backends/s3)
- [Atlantis — GitOps for Terraform](https://www.runatlantis.io/)
- [Terraform Module Registry](https://registry.terraform.io/browse/modules)

---

_Follow my journey! Feel free to ⭐ this repository to stay updated._
