![Progress](https://img.shields.io/badge/Progress-35%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 14 — Hands-On CI Pipeline: CodeBuild + CodePipeline + GitHub

## 📝 Topic: Automating a Docker Build for a Python Flask App, End to End
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 13, 2026

---

## 🎯 Learning Objectives
* Build a real CI pipeline: GitHub push → CodePipeline → CodeBuild → Docker Hub.
* Understand the `buildspec.yaml` file as the heart of the CI process.
* Configure a CodeBuild project's environment, service role, and Privileged Mode correctly.
* Store and reference secrets securely via AWS Systems Manager Parameter Store.
* Debug the most common CodeBuild failure modes: IAM permissions, path errors, Docker errors.
* Connect CodePipeline to GitHub using a GitHub Version 2 connection.

---

## 📁 Part 1 — Project Overview

### The Goal

```
Automate the BUILD process for a simple Python Flask application
```

### Tools Used

| Tool | Role |
|---|---|
| **AWS CodePipeline** | Orchestrator — triggers and sequences the pipeline |
| **AWS CodeBuild** | Build service — actually builds the Docker image |
| **GitHub** | Source repository (used instead of CodeCommit, per Day 12's conclusion) |
| **AWS Systems Manager Parameter Store** | Secret management for Docker Hub credentials |

### The Workflow

```
Code change pushed to GitHub
        ↓
AWS CodePipeline triggers
        ↓
AWS CodeBuild builds a Docker image
        ↓
Image pushed to Docker Hub
```

---

## 📂 Part 2 — Repository Preparation

```
Project repository contains:
  → Flask application code
  → requirements.txt          (Python dependencies)
  → Dockerfile                 (image build instructions)
  → buildspec.yaml             (THE heart of the CI process)
```

### `buildspec.yaml` — The Core CI Definition

```yaml
version: 0.2

env:
  parameter-store:
    DOCKERHUB_USERNAME: /myapp/dockerhub/username
    DOCKERHUB_PASSWORD: /myapp/dockerhub/password
    DOCKERHUB_REGISTRY: /myapp/dockerhub/registry

phases:
  install:
    runtime-versions:
      python: 3.11
    commands:
      - pip install -r requirements.txt

  build:
    commands:
      - echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
      - docker build -t $DOCKERHUB_REGISTRY/flask-app:latest .

  post_build:
    commands:
      - docker push $DOCKERHUB_REGISTRY/flask-app:latest
```

```
Phases defined:
  install     → set up the runtime and install dependencies
  build        → docker login + docker build
  post_build   → docker push the finished image
```

> **Why `buildspec.yaml` is described as "the heart of CI":** it's the single file that fully defines what CodeBuild actually DOES — every phase, every command, every secret reference lives here. Getting this file right is 90% of getting the CI pipeline right.

---

## ⚙️ Part 3 — AWS CodeBuild Configuration

### Environment

```
Managed image: Ubuntu
Runtime: Python
```

### Service Role

```
An IAM Role is REQUIRED for CodeBuild to interact
with other AWS services on its own behalf
  (e.g., reading from Parameter Store, writing logs to CloudWatch)

→ Same Role-based access pattern from IAM (Day 02):
  CodeBuild is a SERVICE, not a person, so it authenticates
  via a Role rather than a User
```

### Privileged Mode — Critical for Docker Builds

```
The "Privileged" checkbox in project environment settings
MUST be enabled for Docker builds to work at all.

Why: building a Docker image requires the Docker DAEMON
     to be running INSIDE the build container.
     Without Privileged Mode, the container lacks the
     necessary permissions to run its own Docker daemon,
     and `docker build` fails outright.
```

> **This is the single most commonly missed setting in this project** — everything else can be configured perfectly, and a Docker build will still fail immediately if Privileged Mode isn't checked. Worth treating as a mandatory checklist item specifically whenever a `buildspec.yaml` includes `docker build`.

---

## 🔐 Part 4 — Security & Secret Management

### Storing Docker Hub Credentials

```
Sensitive values needed:
  → Docker Hub username
  → Docker Hub password
  → Docker Hub registry URL

Stored in: AWS Systems Manager Parameter Store
  → As SECURE STRINGS (encrypted at rest)
```

### Referencing Secrets in `buildspec.yaml`

```yaml
env:
  parameter-store:
    DOCKERHUB_USERNAME: /myapp/dockerhub/username
    DOCKERHUB_PASSWORD: /myapp/dockerhub/password
    DOCKERHUB_REGISTRY: /myapp/dockerhub/registry
```

> **Why Parameter Store instead of hard-coding credentials in the buildspec:** hard-coded secrets in a version-controlled file are a textbook security failure — anyone with repo read access would have live Docker Hub credentials. Parameter Store keeps the actual secret values OUT of the repository entirely; the buildspec only references parameter NAMES, and CodeBuild resolves the actual values at runtime via its IAM Role's permissions.

---

## 🐛 Part 5 — Debugging & Common Issues

### IAM Permissions

```
Problem: CodeBuild fails trying to read from Parameter Store

Fix: The CodeBuild SERVICE ROLE needs explicit SSM
     (Systems Manager) permissions attached
     — e.g., ssm:GetParameters on the specific parameter paths used
```

### Path Errors

```
Problem: build fails because requirements.txt "isn't found"

Fix: Ensure buildspec.yaml's commands correctly point to
     the DIRECTORY actually containing requirements.txt
     — a mismatch between the buildspec's working directory
       assumption and the actual repo structure is a common cause
```

### Docker Errors

```
Two specific gotchas flagged:

1. `docker build` command MUST include the current
   directory as the build context:
     docker build -t myimage:latest .
                                     ↑ don't forget this

2. `docker login` MUST be present in the buildspec
   BEFORE the `docker push` step
     — pushing without first authenticating fails immediately
```

---

## 🔗 Part 6 — Orchestration with AWS CodePipeline

### Connecting to GitHub

```
CodePipeline connects to the GitHub repository using a
GitHub VERSION 2 connection
  → A more modern, secure connection type compared to
    older webhook-based/OAuth token integrations
```

### The Trigger

```
Any code push to the MAIN branch:
  → Automatically triggers the pipeline
  → Pipeline automatically invokes CodeBuild
  → No manual intervention required from this point on
```

---

## ✅ Part 7 — Conclusion

```
End-to-end result achieved:

  Every code push to GitHub
        ↓
  Automated Docker image BUILD
        ↓
  Automated PUSH to Docker Hub

  → Completes the full Continuous Integration cycle
    for this Flask application
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **`buildspec.yaml`** | The YAML file defining CodeBuild's install/build/post_build phases and commands |
| **CodeBuild Service Role** | The IAM Role CodeBuild assumes to access other AWS services on its own behalf |
| **Privileged Mode** | A CodeBuild environment setting required to run the Docker daemon inside the build container |
| **AWS Systems Manager Parameter Store** | A service for securely storing configuration values and secrets (as Secure Strings) |
| **Secure String** | An encrypted parameter type in Parameter Store, used for sensitive values like passwords |
| **`env: parameter-store`** | The buildspec section mapping environment variable names to Parameter Store paths |
| **GitHub Version 2 Connection** | A modern, secure connection type for integrating CodePipeline with a GitHub repository |
| **Docker Build Context** | The directory (often `.`) Docker uses to locate files referenced in the Dockerfile |

---

## 📂 Summary of Tasks
- ✅ Prepared: A repository with a Flask app, `requirements.txt`, `Dockerfile`, and `buildspec.yaml`.
- ✅ Understood: `buildspec.yaml`'s install/build/post_build phase structure as the core of the CI definition.
- ✅ Configured: A CodeBuild project with an Ubuntu managed image and Python runtime.
- ✅ Attached: An IAM Service Role enabling CodeBuild to access other AWS services.
- ✅ Enabled: Privileged Mode — the critical, easy-to-miss setting required for Docker builds.
- ✅ Stored: Docker Hub credentials securely in Parameter Store as Secure Strings.
- ✅ Referenced: Those secrets in `buildspec.yaml` via the `env: parameter-store` section.
- ✅ Debugged: IAM permission gaps, path errors, and Docker build/login command mistakes.
- ✅ Connected: CodePipeline to GitHub via a GitHub Version 2 connection, triggering on pushes to `main`.
- ✅ Verified: A full end-to-end CI cycle — GitHub push → Docker build → Docker Hub push, fully automated.

---

## 💡 My Takeaway

Privileged Mode is the detail I most want to remember from this session — it's exactly the kind of single checkbox that's easy to overlook, produces a confusing failure with no obvious connection to "I forgot a checkbox," and yet is completely mandatory the moment `docker build` shows up in a buildspec. I want to treat "is Privileged Mode enabled?" as a standing first question whenever a CodeBuild Docker step fails mysteriously, the same way I now treat "check the Security Group" as a first question for unreachable services.

The Parameter Store pattern for secrets is a clean, concrete example of something I've been reasoning about abstractly since the Kubernetes ConfigMaps/Secrets session — keep sensitive values OUT of version-controlled files, reference them by name, and let the runtime (CodeBuild here, kubelet there) resolve the actual value via its own permissions. Same principle, different platform, same reason for existing.

This project also directly connects to things I've already built — the Docker build + push mechanics here are exactly what I'd want wired around Intervio or GrowEasy's deployment, replacing manual `docker build && docker push` commands with something that fires automatically on every GitHub push to `main`. That's a genuinely actionable next step for my own portfolio projects, not just an abstract exercise.

---


## 🔗 Resources
* [AWS CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
* [Buildspec Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
* [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
* [CodePipeline GitHub Version 2 Connections](https://docs.aws.amazon.com/dtconsole/latest/userguide/welcome-connections.html)
* [Docker Build Reference](https://docs.docker.com/engine/reference/commandline/build/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*