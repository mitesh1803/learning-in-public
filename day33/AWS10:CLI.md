![Progress](https://img.shields.io/badge/Progress-33%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 10 — AWS CLI Deep Dive

## 📝 Topic: Automating Infrastructure Management from the Command Line
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 11, 2026

---

## 🎯 Learning Objectives
* Understand why the AWS Management Console doesn't scale for automation-heavy workflows.
* Understand how the CLI, Terraform, and CloudFormation all sit as abstraction layers over raw AWS APIs.
* Install and verify the AWS CLI.
* Configure the CLI to authenticate against a real AWS account using IAM credentials.
* Run practical commands to list and create AWS resources from the terminal.
* Reinforce the "never use root" principle in a CLI-specific context.

---

## ❓ Part 1 — Why Use the AWS CLI?

### The UI's Limitation

```
AWS Management Console (the web UI):
  → Great for learning, exploring, one-off tasks
  → TERRIBLE for automation

Example of the pain point:
  Manually creating 10 VPCs, or 20 S3 buckets,
  by clicking through the console one at a time
  → Time-consuming
  → Error-prone (easy to misconfigure one out of twenty)
  → Impossible to reliably repeat or version-control
```

### The API Alternative

```
AWS exposes essentially everything the Console can do
via APIs (Application Programming Interfaces).

  → Every button click in the Console ultimately
    corresponds to an underlying API call
```

### Abstraction Layers Over Raw APIs

```
Working with raw APIs directly means:
  → Manually constructing HTTP requests
  → Handling authentication signing
  → Parsing raw JSON responses yourself

Tools that abstract this away:
  AWS CLI          → simple commands, CLI handles the API details
  Terraform (IaC)   → declarative config files, provider handles APIs
  CloudFormation    → AWS-native declarative templates (JSON/YAML)
```

### When to Use Which Tool

| Tool | Best for |
|---|---|
| **AWS CLI** | Quick, day-to-day tasks — e.g. `aws s3 ls` to check something fast |
| **Terraform / CloudFormation** | Large, complex, version-controlled infrastructure stacks managed as code |

> **The distinction that matters:** the CLI is for *ad hoc, interactive* operations — check something, create one resource, debug something quickly. Terraform/CFT are for *repeatable, declared* infrastructure that needs to be reviewed, versioned, and reproduced reliably. Using the CLI to hand-build a 20-resource production stack defeats the purpose — that's exactly the scenario Infrastructure as Code exists for.

---

## 🛠️ Part 2 — Installation & Configuration

### Installation

```
Download the official installer for your OS from the AWS website:
  → macOS
  → Linux
  → Windows
```

```
Tips for Windows users specifically:
  → Consider Oracle VirtualBox running a Linux distribution, OR
  → Use Git Bash for a more Unix-like CLI experience
    (avoids some Windows-specific path/quoting quirks)
```

```bash
# Verify installation succeeded
aws --version
# aws-cli/2.x.x Python/3.x.x ...
```

### Configuration

```bash
aws configure
```

```
Prompts requested:
  AWS Access Key ID       ← obtained via IAM user's Security Credentials
  AWS Secret Access Key   ← obtained via IAM user's Security Credentials
  Default region name     → e.g., us-east-1
  Default output format   → JSON recommended
```

> **Where credentials come from:** IAM Console → Users → (your user) → Security Credentials tab → Create Access Key. This directly reuses the IAM user creation flow from AWS Day 02 — the CLI authenticates as whichever IAM identity these keys belong to, with exactly the permissions that identity's attached policies/groups grant.

---

## 💻 Part 3 — Practical Demos

### Listing Resources

```bash
aws s3 ls
# 2026-06-01  my-app-bucket
# 2026-06-15  backup-bucket
```

> A single command replaces navigating into the S3 console, waiting for the page to load, and scanning the bucket list visually — the value compounds massively once this becomes part of a script instead of a one-off check.

### Creating Resources

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0
```

```
Command specifies:
  → AMI ID              (which OS image to launch)
  → Instance type        (t2.micro, etc.)
  → Key pair             (for SSH access)
  → Security configuration (Security Group + subnet)
```

### Troubleshooting

```bash
aws ec2 run-instances --instance-type t2.micro
# Error: Missing required parameter: image-id
```

> **Why CLI errors are genuinely useful for debugging:** unlike a Console form that might just leave a field looking wrong, the CLI immediately and explicitly names the exact missing/invalid parameter — this makes command construction more of a fast iterate-and-fix loop than a guessing game.

---

## ✅ Part 4 — Key Takeaways for DevOps Engineers

```
1. Don't use root user
   → Always use IAM users (with appropriate permissions)
     for CLI authentication, exactly as covered on Day 02
   → Root access keys configured in a local CLI profile
     are an especially dangerous credential to have sitting
     on a laptop or in a script

2. Use Documentation
   → The AWS CLI Reference docs define every command's
     exact structure, required/optional arguments, and examples
   → Essential for anything beyond the handful of commands
     memorized from a tutorial

3. Automation Mindset
   → The real goal isn't "learn CLI commands" —
     it's shifting from clicking through a Console
     to scripting and eventually full Infrastructure as Code
   → The CLI is the first, most direct step in that
     progression — good for day-to-day tasks, and also
     the foundation that makes shell scripting AWS
     operations possible at all
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS Management Console** | The web-based UI for interacting with AWS — not automation-friendly |
| **AWS API** | The underlying programmatic interface every AWS interaction (Console or CLI) ultimately goes through |
| **AWS CLI** | A command-line tool abstracting raw API calls into simple commands |
| **Terraform** | A third-party Infrastructure as Code tool for declarative, version-controlled infrastructure |
| **CloudFormation (CFT)** | AWS's native Infrastructure as Code service, using JSON/YAML templates |
| **`aws configure`** | The command that links the local CLI to an AWS account via IAM credentials |
| **Access Key ID / Secret Access Key** | The credential pair used to authenticate CLI requests as a specific IAM identity |
| **`aws s3 ls`** | Lists S3 buckets in the authenticated account |
| **`aws ec2 run-instances`** | Launches a new EC2 instance from the command line |

---

## 📂 Summary of Tasks
- ✅ Understood: Why the Console doesn't scale for repetitive/automated infrastructure tasks.
- ✅ Understood: How the CLI, Terraform, and CloudFormation all abstract raw AWS APIs differently.
- ✅ Learned: When to reach for the CLI (quick tasks) vs. Terraform/CFT (large, versioned stacks).
- ✅ Installed: The AWS CLI and verified it via `aws --version`.
- ✅ Configured: The CLI using `aws configure` with IAM Access Key ID/Secret Access Key.
- ✅ Practiced: Listing S3 buckets via `aws s3 ls`.
- ✅ Practiced: Constructing an `aws ec2 run-instances` command with the required parameters.
- ✅ Observed: How CLI error messages explicitly identify missing/invalid parameters for fast debugging.
- ✅ Reinforced: Using IAM users (never root) for CLI authentication.

---

## 💡 My Takeaway

The CLI-vs-Terraform/CFT framing gave me a cleaner mental model than I had going in — I'd been treating "CLI vs Infrastructure as Code" as a binary choice, but it's really a spectrum by USE CASE: the CLI for a quick check or a one-off resource, Terraform for anything that needs to be reproducible, reviewable, and version-controlled. That reframes when I should actually reach for `commit-gen` and `.tf` files in my own projects versus just running a quick `aws` command directly.

Configuring `aws configure` with IAM Access Key credentials was a nice direct callback to the IAM session from Day 02 — the CLI isn't a separate authentication system, it's just another consumer of the exact same IAM identity, with exactly the permissions that identity's policies grant. A CLI profile configured with an over-permissioned IAM user is just as risky as an over-permissioned Console login — same principle, different interface.

The explicit "missing required parameter" error behavior is a genuinely underrated CLI advantage worth remembering — it turns command construction into a fast, self-correcting loop instead of trial-and-error against a UI form. Worth leaning on `--help` and the CLI Reference docs early rather than guessing flag names from memory.

---

## 🔗 Resources
* [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
* [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
* [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [IAM Access Keys Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)
* [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*