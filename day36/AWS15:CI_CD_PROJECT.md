![Progress](https://img.shields.io/badge/Progress-36%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 15 — AWS CodeDeploy: Completing the CI/CD Pipeline

## 📝 Topic: Continuous Delivery to EC2 — CodeDeploy Agent, AppSpec, and Full Pipeline Integration
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 14, 2026
---

## 🎯 Learning Objectives
* Build the CD (Continuous Delivery) half of the pipeline, completing what CodeBuild started on Day 14.
* Register a CodeDeploy Application and configure an EC2 target with the correct IAM Role.
* Install and understand the role of the CodeDeploy Agent on the target instance.
* Create a Deployment Group using Tags to identify target instances.
* Understand `appspec.yml` as CodeDeploy's equivalent of `buildspec.yaml`.
* Write deployment lifecycle scripts to pull and run a Docker container.
* Debug the most common CodeDeploy failure: port conflicts from a still-running previous container.
* Integrate CodeDeploy as a new stage inside the existing CodePipeline.

---

## 🎯 Part 1 — Objective and Architecture

### Where This Fits

```
Day 14 built the CI half:
  GitHub push → CodePipeline → CodeBuild → Docker image → Docker Hub

Day 15 builds the CD half:
  ...Docker Hub → CodeDeploy → pulls image → runs it on EC2
```

### Full Pipeline Architecture

```
AWS CodePipeline (orchestrator)
      ├── Stage 1: Source   → GitHub
      ├── Stage 2: Build    → AWS CodeBuild  (CI)
      └── Stage 3: Deploy   → AWS CodeDeploy (CD)
```

> **The conceptual throughline:** CodePipeline remains the single orchestrator across the whole series — Day 13 introduced it, Day 14 wired in CodeBuild as its CI stage, and today adds CodeDeploy as its CD stage. Nothing about the orchestrator itself changes; each day simply plugs in the next stage.

---

## 🛠️ Part 2 — AWS CodeDeploy Setup Steps

### Step 1: Application Creation

```
AWS CodeDeploy Console → Create Application
  → Register a new "Application" — this is the top-level
    container CodeDeploy uses to organize deployments
```

### Step 2: EC2 Configuration

```
Launch an EC2 instance (Ubuntu)

Critical requirement: an IAM ROLE attached to the instance
  → Allows the instance to interact with the CodeDeploy service
  → Same "Roles for non-human access" pattern from
    IAM (Day 02) — the EC2 instance itself needs to
    authenticate to AWS, so it uses a Role, not a User
```

### Step 3: Agent Installation

```
Install the CodeDeploy AGENT on the EC2 instance:
  → A background process that runs ON the instance
  → Enables communication BETWEEN the instance and
    the CodeDeploy service

Without the agent running:
  → CodeDeploy has no way to actually deliver
    deployment instructions to that instance —
    the instance is invisible to CodeDeploy, even
    if it's otherwise perfectly configured
```

```bash
# General installation pattern (Ubuntu)
sudo apt update
sudo apt install ruby-full wget -y
cd /home/ubuntu
wget https://aws-codedeploy-<region>.s3.<region>.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo service codedeploy-agent status
```

### Step 4: Deployment Group

```
Create a DEPLOYMENT GROUP:
  → Defines WHICH instances are the deployment TARGET
  → Uses TAGS to identify matching EC2 instances
    (e.g., Key: Environment, Value: Production)

  → CodeDeploy queries instances by tag at deployment
    time — any instance carrying the matching tag(s)
    becomes part of that deployment group automatically
```

---

## 📄 Part 3 — Deployment Configuration

### The `appspec.yml` File

```
Required at the ROOT of the GitHub repository —
tells CodeDeploy exactly HOW to handle the deployment.

This is CodeDeploy's equivalent of buildspec.yaml
from CodeBuild (Day 14) — same core idea, different
service and different lifecycle hooks.
```

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/app

hooks:
  BeforeInstall:
    - location: scripts/stop_container.sh
      timeout: 300
  AfterInstall:
    - location: scripts/start_container.sh
      timeout: 300
```

### Deployment Scripts

```bash
# stop_container.sh
#!/bin/bash
docker stop flask-app-container || true
docker rm flask-app-container || true
```

```bash
# start_container.sh
#!/bin/bash
docker pull mydockerhubuser/flask-app:latest
docker run -d --name flask-app-container -p 80:5000 mydockerhubuser/flask-app:latest
```

> **Why two separate scripts, tied to specific lifecycle hooks:** `BeforeInstall` runs the STOP script — cleaning up whatever was running previously — before `AfterInstall` runs the START script that pulls the newest image and launches it. Getting this ORDER right (via the correct hook names) is what prevents the port-conflict failure covered next.

---

## 🐛 Part 4 — Troubleshooting & Debugging

### The Common Failure: Port Conflicts

```
Symptom: deployment fails during the start phase

Root cause: a PREVIOUS container is still running,
            already bound to the same port
            (e.g., port 80 already in use)

  → docker run fails outright because the port
    is unavailable — the new container can never start
    while the old one still holds that port
```

### The Fix

```
Ensure stop_container.sh THOROUGHLY cleans up:
  → docker stop <container-name>
  → docker rm <container-name>

  → Both steps matter: stopping alone leaves a
    stopped-but-still-named container that can
    still block a `docker run --name` using the
    same container name
```

### Verification

```bash
# On the EC2 instance, confirm the new container is running
sudo docker ps

# CONTAINER ID   IMAGE                          STATUS         PORTS
# a1b2c3d4e5f6   mydockerhubuser/flask-app:latest  Up 2 minutes   0.0.0.0:80->5000/tcp
```

> **The recurring pattern here, once again:** this is functionally the same class of bug as the Security Group / Privileged Mode issues from earlier sessions — something that LOOKS configured correctly but fails because of one specific, easy-to-miss cleanup/permission step. Treating "did the previous container actually get stopped AND removed?" as a standard check before assuming a deeper problem.

---

## 🔗 Part 5 — Pipeline Integration

### Adding the CodeDeploy Stage

```
Existing CodePipeline (Source → Build) gets a new stage:
  → Add Stage → Deploy
  → Provider: AWS CodeDeploy
  → Select the Application and Deployment Group
    created in Part 2
```

### Testing End to End

```
CodePipeline Console → "Release Change" button
  → Manually re-triggers the ENTIRE pipeline from scratch
  → Confirms Source → Build → Deploy all execute successfully
    in sequence, without needing a fresh GitHub push
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS CodeDeploy** | AWS's managed service automating code/container deployments to EC2, Lambda, or ECS |
| **CodeDeploy Agent** | A background process on the target instance enabling communication with the CodeDeploy service |
| **Deployment Group** | Defines the target instances for a deployment, typically identified via Tags |
| **`appspec.yml`** | The file at the repo root defining CodeDeploy's deployment lifecycle hooks and file locations |
| **Lifecycle Hooks** | Named stages (`BeforeInstall`, `AfterInstall`, etc.) in `appspec.yml` mapping to specific scripts |
| **`docker stop` / `docker rm`** | Commands required together to fully clean up a container before starting a replacement |
| **Release Change** | The CodePipeline Console button that manually re-triggers a full pipeline run |

---

## 📂 Summary of Tasks
- ✅ Understood: How CodeDeploy completes the CD half of the pipeline started with CodeBuild on Day 14.
- ✅ Registered: A new Application in the CodeDeploy console.
- ✅ Configured: An EC2 instance with the IAM Role required for CodeDeploy interaction.
- ✅ Installed: The CodeDeploy Agent on the target instance.
- ✅ Created: A Deployment Group using Tags to identify the target EC2 instance(s).
- ✅ Wrote: An `appspec.yml` with `BeforeInstall`/`AfterInstall` hooks.
- ✅ Wrote: `stop_container.sh` and `start_container.sh` scripts to manage the Docker container lifecycle.
- ✅ Debugged: A port-conflict failure caused by an incompletely cleaned-up previous container.
- ✅ Verified: The running container via `sudo docker ps`.
- ✅ Integrated: CodeDeploy as a new stage in the existing CodePipeline.
- ✅ Tested: The full end-to-end pipeline using the "Release Change" button.

---

## 💡 My Takeaway

The port-conflict debugging story is the part of today's session I'll remember longest — it's a perfect example of a failure that looks like "CodeDeploy is broken" but is actually just "the stop script didn't fully clean up." `docker stop` alone isn't enough if the container name still exists; `docker rm` has to follow it. That's a small, very specific detail, but it's exactly the kind of thing that costs real debugging time in a live deployment if it's not already a known gotcha going in.

Seeing `appspec.yml`'s `BeforeInstall`/`AfterInstall` hooks side by side with `buildspec.yaml`'s `install`/`build`/`post_build` phases from Day 14 made the CI/CD symmetry click clearly — CodeBuild and CodeDeploy each have their own YAML contract defining "what happens, in what order," just with different lifecycle vocabulary suited to their different jobs (building vs. deploying).

With this session, the full pipeline from Days 13–15 is genuinely complete end to end: a GitHub push now results in an automated build, image push, AND deployment to a running EC2 instance, with zero manual steps in between. That's the first fully closed-loop CI/CD pipeline I've built in this series, and it's a natural template to adapt directly onto Intervio or GrowEasy's actual deployment process.

---


## 🔗 Resources
* [AWS CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
* [CodeDeploy AppSpec File Reference](https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file.html)
* [CodeDeploy Agent Installation](https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install.html)
* [CodeDeploy Deployment Groups](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-groups.html)
* [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*