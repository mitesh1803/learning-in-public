![Progress](https://img.shields.io/badge/Progress-34%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 12 — AWS CodeCommit & the AWS CI/CD Ecosystem

## 📝 Topic: Native AWS Source Control, Setup, Best Practices & Its Real-World Limitations
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 12, 2026

---

## 🎯 Learning Objectives
* Understand AWS's native CI/CD service suite and how each piece maps to familiar open-source tools.
* Understand what AWS CodeCommit is and why organizations choose it over self-hosted Git.
* Set up an IAM user correctly for CodeCommit access, avoiding the root account.
* Understand CodeCommit's real-world limitations, and why this series pivots to GitHub for the rest of the CI/CD track.
* Clone, commit to, and push into a private CodeCommit repository hands-on.

---

## 🔧 Part 1 — The AWS CI/CD Ecosystem

### AWS's Native Alternative to Open-Source Tools

```
Instead of Jenkins, GitLab CI, or Argo CD,
AWS provides its own fully-managed CI/CD suite:
```

| AWS Service | Role | Familiar Equivalent |
|---|---|---|
| **AWS CodeCommit** | Version control, hosts private repositories | GitHub / GitLab |
| **AWS CodePipeline** | Orchestrates the end-to-end delivery workflow | Jenkins Pipelines |
| **AWS CodeBuild** | Compiles source code, runs tests, produces packages | Maven / Docker builds |
| **AWS CodeDeploy** | Automates deployments to EC2 or Kubernetes | Argo CD / Ansible |

> **The strategic value of this mapping:** every one of these AWS services solves the exact same category of problem as a tool likely already familiar from the DevOps world — the mental model transfers directly, only the specific configuration syntax and AWS-native integration points change.

---

## 📂 Part 2 — What Is AWS CodeCommit?

### Definition

```
CodeCommit = a secure, highly scalable, fully managed
             source control service hosting PRIVATE Git repositories
```

### Why Organizations Choose It

```
Alternative: self-host a GitLab instance or private
             Enterprise Git server

  → Requires patching, scaling, and ongoing
    operational overhead — the exact same
    "manage your own infrastructure" burden
    covered all the way back on AWS Day 01

CodeCommit's pitch:
  → Scales AUTOMATICALLY as team size or code volume grows
  → No servers to patch or scale manually — AWS manages it
```

### Strictly Private

```
Unlike GitHub or public GitLab instances:
  CodeCommit repositories are COMPLETELY PRIVATE
  within an AWS organization — they CANNOT be exposed
  publicly, period.
```

> This is a meaningful architectural constraint, not just a default setting — if a public-facing open-source project is the goal, CodeCommit is fundamentally the wrong tool from the outset, regardless of configuration.

---

## 🛠️ Part 3 — Setup and Best Practices

### Avoid Root Accounts

```
Using the root account for CodeCommit is HIGHLY DISCOURAGED:
  → Permission and security limitations apply specifically
    to SSH/HTTPS Git operations under root
  → This is the same "never use root for daily work"
    principle from IAM (AWS Day 02), applied here concretely
```

### IAM User Authentication

```
Required setup:
  1. Create a dedicated IAM user
  2. Attach the AWSCodeCommitPowerUser managed policy

  This grants:
    → Full repository access (clone, push, pull, manage)
    → Integration access to related services:
        - CloudWatch (monitoring repo activity)
        - Amazon SNS (notifications on repo events)
```

> Using a managed policy here (rather than writing a custom one from scratch) mirrors the exact same AWS Managed Policy pattern from the IAM session — reach for a pre-built policy covering the standard use case before reaching for custom JSON.

---

## ⚠️ Part 4 — Limitations of CodeCommit

```
Even in a 100%-AWS shop, many companies still stick with
GitHub or GitLab instead of CodeCommit. Why:
```

### Fewer Advanced Features

```
CodeCommit lacks modern developer conveniences like:
  → GitHub Copilot integration
  → Interactive cloud-based editing environments
  → Extensive, mature, third-party automated DevOps tooling
```

### Ecosystem Rigidity

```
CodeCommit's integrations are mostly constrained to
the AWS ecosystem itself
  → Wiring up multi-cloud or third-party pipelines
    becomes noticeably harder compared to GitHub's
    broad, mature integration marketplace
```

> **Why this matters for the rest of this learning path:** because CodeCommit is LESS commonly requested in industry interviews and real job postings compared to GitHub-based workflows, the instructor explicitly pivots the remainder of the CI/CD track to integrating GitHub with CodeBuild, CodeDeploy, and CodePipeline instead. CodeCommit is worth understanding conceptually, but GitHub is where the practical, job-relevant depth will be built.

---

## 🏋️ Part 5 — Hands-On Assignment

### Step 1: IAM User + Policy

```
Create an IAM user
Attach: AWSCodeCommitPowerUser managed policy
```

### Step 2: Generate HTTPS Git Credentials

```
IAM Console → Users → (your user) → Security Credentials
  → HTTPS Git credentials for AWS CodeCommit
  → Generate a username/password pair specifically for Git operations
```

### Step 3: Clone the Repository

```bash
git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/<repo-name>
```

### Step 4: Configure Git Identity

```bash
git config --global user.name "Mitesh"
git config --global user.email "your-email@example.com"
```

### Step 5: Commit and Push

```bash
echo "test file for CodeCommit" > test.txt
git add test.txt
git commit -m "Initial commit to CodeCommit"
git push
```

> This walkthrough is functionally identical to the GitHub push workflow from the earlier EC2 static site deployment project — same `git init`/`add`/`commit`/`push` mechanics, just pointed at a CodeCommit HTTPS URL instead of a GitHub remote. The Git fundamentals transfer completely; only the remote and the credential type (HTTPS Git credentials vs. a GitHub personal access token) change.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS CodeCommit** | AWS's fully managed, private Git repository hosting service |
| **AWS CodePipeline** | Orchestrates the end-to-end CI/CD delivery workflow |
| **AWS CodeBuild** | Compiles code, runs tests, and produces build artifacts |
| **AWS CodeDeploy** | Automates deployment of built artifacts to EC2, Kubernetes, or other targets |
| **`AWSCodeCommitPowerUser`** | The AWS Managed Policy granting full CodeCommit repo access plus related service integrations |
| **HTTPS Git Credentials** | A special username/password pair generated per IAM user specifically for authenticating Git operations against CodeCommit |
| **Ecosystem Rigidity** | The limitation where a tool's integrations are mostly confined to its native platform, making outside integrations harder |

---

## 📂 Summary of Tasks
- ✅ Mapped: AWS's native CI/CD suite (CodeCommit, CodePipeline, CodeBuild, CodeDeploy) to familiar open-source equivalents.
- ✅ Understood: What CodeCommit is and why it removes the operational burden of self-hosting Git.
- ✅ Learned: CodeCommit repositories are strictly private and can never be made public.
- ✅ Set up: An IAM user with the `AWSCodeCommitPowerUser` managed policy, avoiding the root account.
- ✅ Understood: CodeCommit's real limitations — fewer modern dev features, ecosystem rigidity.
- ✅ Noted: The rest of this CI/CD track will pivot to GitHub integration, since it's more industry-relevant.
- ✅ Practiced: Generating HTTPS Git credentials, cloning, committing, and pushing to a CodeCommit repo.

---

## 💡 My Takeaway

The most useful thing about today's session wasn't really CodeCommit itself — it was the instructor's honesty about when NOT to use it. Explicitly saying "this is less commonly asked about in interviews, so we're pivoting the rest of this track to GitHub" is a genuinely useful signal about where to actually invest depth. It's a good reminder that "AWS has a native service for X" doesn't automatically mean that native service is the industry-standard choice worth mastering — sometimes the native option is worth understanding conceptually (for breadth, and for the rare all-AWS shop that does use it) without needing deep hands-on mastery.

The service-mapping table (CodeCommit↔GitHub, CodePipeline↔Jenkins, CodeBuild↔Maven/Docker, CodeDeploy↔Argo CD/Ansible) is genuinely useful shorthand for interviews — being able to say "AWS's native CI/CD suite maps roughly onto [familiar tool]" for each piece shows both AWS-specific knowledge and the ability to reason about the underlying CI/CD concepts independent of any one vendor's naming.

Given my own portfolio (Intervio, ATS Shortlister, GrowEasy) are already on GitHub, this session mostly confirms the right call was already made there — and sets up the next few sessions (GitHub + CodeBuild + CodeDeploy + CodePipeline) as directly relevant to actually wiring a real CI/CD pipeline around those existing projects.

---


## 🔗 Resources
* [AWS CodeCommit Documentation](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)
* [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
* [AWS CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
* [AWS CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
* [CodeCommit HTTPS Git Credentials Setup](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_ssh-keys.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*