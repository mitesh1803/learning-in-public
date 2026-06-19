![Progress](https://img.shields.io/badge/Progress-14%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 20 — GitHub Actions & CI/CD Pipelines

## 📝 Topic: GitHub Actions as a CI/CD Solution — Concepts, Hands-On & Jenkins Comparison
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 16, 2026

---

## 🎯 Learning Objectives
* Understand what GitHub Actions is and where it fits in the CI/CD landscape.
* Know why GitHub Actions is ideal for GitHub-native teams — and where it falls short.
* Understand how pipelines are defined using YAML in `.github/workflows/`.
* Configure multiple workflows for different purposes — PR validation, linting, CI, deployment.
* Run a real Python unit test pipeline from scratch.
* Understand the difference between GitHub-hosted and self-hosted runners.
* Manage secrets natively within GitHub for sensitive values like kubeconfig and tokens.
* Make an informed decision between GitHub Actions and Jenkins based on team context.

---

## 💡 Part 1 — What is GitHub Actions?

> *"GitHub Actions is a CI/CD tool deeply integrated into the GitHub platform."*

Unlike Jenkins, CircleCI, or TeamCity — which are standalone CI/CD tools you connect to your repo — GitHub Actions lives **inside GitHub itself**. There is no separate server to install, no webhook to configure, no third-party integration to manage.

```
Traditional CI/CD setup:
  GitHub repo → webhook → Jenkins server → run pipeline → report back to GitHub

GitHub Actions setup:
  GitHub repo → pipeline defined in .github/workflows/ → runs on GitHub's infrastructure
```

The pipeline is code. It lives in the repository. It versions with the repository.

### 🎯 Who it's built for

GitHub Actions is purpose-built for teams that live in the GitHub ecosystem. If your source control, code review, issue tracking, and collaboration all happen on GitHub, Actions removes every layer of integration overhead.

> ⚠️ **The trade-off worth knowing upfront:** GitHub Actions is platform-dependent. If your organization migrates from GitHub to GitLab, Bitbucket, or Azure DevOps later, every workflow file needs to be rewritten from scratch. Compare this to Terraform — which is intentionally cloud-agnostic. This is the same lock-in trade-off, applied to CI/CD.

---

## 📁 Part 2 — Getting Started: Project Structure

Pipelines in GitHub Actions are YAML files stored in a specific directory:

```
your-repo/
├── .github/
│   └── workflows/
│       ├── ci.yml          ← runs tests on every push/PR
│       ├── lint.yml        ← linting checks on PRs
│       ├── deploy.yml      ← deployment on merge to main
│       └── pr-check.yml    ← PR validation gates
├── src/
├── tests/
└── README.md
```

**You can have multiple workflows** — each file is an independent pipeline triggered by its own events. There's no limit.

### 🔑 Core YAML Concepts

```yaml
name: CI Pipeline          # display name in GitHub UI

on:                        # trigger — what causes this to run
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:                      # what to actually do
  test:                    # job name (arbitrary)
    runs-on: ubuntu-latest # the runner environment

    steps:                 # ordered list of actions
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: python -m pytest tests/
```

### 📋 Key YAML Fields

| Field | What it does |
|---|---|
| `name` | Display name shown in GitHub's Actions tab |
| `on` | The trigger — `push`, `pull_request`, `schedule`, `workflow_dispatch` |
| `jobs` | One or more parallel or sequential jobs to run |
| `runs-on` | The runner OS — `ubuntu-latest`, `windows-latest`, `macos-latest` |
| `steps` | Ordered list of actions within a job |
| `uses` | Reference a pre-built action from the Marketplace |
| `run` | Execute a raw shell command |
| `with` | Pass parameters into a `uses` action |

---

## 🧪 Part 3 — Hands-On: Python Unit Test Pipeline

### 🎯 The Goal

Run a Python unit test suite automatically on every push and pull request — without managing any server.

### 📝 The Workflow File

```yaml
# .github/workflows/python-tests.yml

name: Python Unit Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run unit tests
        run: |
          python -m pytest tests/ -v --tb=short
```

### 🔄 What Happens Step by Step

```
Developer opens a Pull Request
        ↓
GitHub detects the PR → finds .github/workflows/python-tests.yml
        ↓
GitHub spins up a fresh ubuntu-latest runner (virtual machine)
        ↓
Step 1: Checkout → pulls the PR branch code onto the runner
        ↓
Step 2: Setup Python → installs Python 3.11 on the runner
        ↓
Step 3: Install dependencies → pip install from requirements.txt
        ↓
Step 4: Run tests → pytest runs → pass ✅ or fail ❌
        ↓
Result shown on the PR — green check or red X
        ↓
Runner is discarded — completely fresh environment next time
```

No server to maintain. No environment drift. Every run starts from zero.

---

## 🖥️ Part 4 — Runners

A **runner** is the machine that executes your workflow steps.

### GitHub-Hosted Runners (Default)

```
ubuntu-latest    → Ubuntu 22.04 LTS
windows-latest   → Windows Server 2022
macos-latest     → macOS 13 (Ventura)
```

GitHub manages these. They are provisioned on demand, discarded after the job, and free for public repositories. For private repos, they consume the monthly minutes quota.

**What comes pre-installed:**
* Docker, Git, Node.js, Python, Java, Go, Ruby
* AWS CLI, Azure CLI, GCloud CLI
* Common build tools (make, cmake, gradle, maven)

### Self-Hosted Runners

For situations where the default runners don't work:

```
Use self-hosted runners when:
  → You need specific hardware (GPU, ARM, high-memory)
  → Your job requires access to internal network resources
  → Security policy requires code to never leave your infrastructure
  → You need more compute minutes than GitHub's quota allows
```

```yaml
# Use a self-hosted runner instead of GitHub's
jobs:
  build:
    runs-on: self-hosted   # ← points to your own registered machine
```

Setting up a self-hosted runner:

```
GitHub repo → Settings → Actions → Runners → New self-hosted runner
→ Follow the registration script for your OS
→ Runner registers itself, waits for jobs
```

---

## 🔐 Part 5 — Secrets Management

Sensitive values — API keys, kubeconfig files, database passwords, tokens — never go in the workflow YAML file directly. GitHub provides native secrets storage.

### Setting a Secret

```
GitHub repo → Settings → Secrets and variables → Actions → New repository secret

Name:   KUBECONFIG
Value:  <paste the full kubeconfig content>
```

### Using a Secret in a Workflow

```yaml
steps:
  - name: Deploy to Kubernetes
    env:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
    run: kubectl apply -f deployment.yaml

  - name: SonarQube scan
    env:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    run: sonar-scanner
```

**How secrets work:**

```
Secret stored in GitHub's encrypted vault
        ↓
Injected as environment variable at runtime
        ↓
Never printed in logs (GitHub redacts them automatically)
        ↓
Never accessible outside the workflow — not in PRs from forks by default
```

### Secret Scopes

| Scope | Where it's available |
|---|---|
| **Repository secret** | Only in that specific repo's workflows |
| **Environment secret** | Only when the workflow targets a specific environment (staging, prod) |
| **Organization secret** | Available across all repos in the organization |

---

## ⚖️ Part 6 — GitHub Actions vs Jenkins

### 🟢 GitHub Actions Advantages

| Factor | GitHub Actions | Jenkins |
|---|---|---|
| **Setup** | Zero — already in GitHub | Install, configure, maintain a server |
| **Maintenance** | GitHub manages it | You manage OS, plugins, updates, security |
| **Hosting** | GitHub's infrastructure | Your server or cloud VM |
| **Cost (public repos)** | Free, unlimited minutes | Server costs (EC2, GCP, etc.) |
| **UI** | Clean, integrated into GitHub | Functional but dated |
| **Marketplace** | 20,000+ pre-built actions | Plugin ecosystem (powerful but complex) |
| **Learning curve** | Low — YAML, GitHub concepts | Higher — Groovy DSL, Jenkins architecture |

### 🔴 GitHub Actions Disadvantages

| Factor | Reality |
|---|---|
| **Platform lock-in** | Workflows only work on GitHub. Migrate to GitLab → rewrite everything |
| **Complex pipelines** | Very large pipelines become hard to read and debug in YAML |
| **Self-hosted burden** | If you need self-hosted runners, you're back to managing servers |
| **Minutes quota** | Private repos have a monthly minutes limit — large orgs hit it |
| **GitOps friction** | Less native integration with Flux/Argo CD compared to GitLab CI |

### 🤔 The Decision Framework

```
Choose GitHub Actions if:
  ✅ Your team is fully on GitHub
  ✅ You don't plan to migrate version control platforms
  ✅ You want zero CI/CD infrastructure to manage
  ✅ Your pipelines are straightforward — test, build, deploy
  ✅ Public repos (it's completely free)

Choose Jenkins if:
  ✅ You're on multiple VCS platforms (GitHub + Bitbucket + internal)
  ✅ You need maximum plugin flexibility and customization
  ✅ Your organization already has Jenkins expertise and infrastructure
  ✅ You need pipelines that run on-premises, fully air-gapped
```

> **The instructor's framing:** GitHub Actions is to CI/CD what AWS CDK is to IaC — powerful and ergonomic within its ecosystem, but a liability if you ever leave it. Terraform (IaC) and Jenkins (CI/CD) are the platform-agnostic alternatives that transfer across environments.

---

## 🔄 Part 7 — Common Workflow Patterns

### PR Validation Gate

```yaml
name: PR Checks

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install flake8 && flake8 src/

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt && pytest tests/
```

Both `lint` and `test` jobs run in **parallel** — PR is blocked until both pass.

### Deploy on Merge to Main

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to EC2
        run: |
          ssh -i key.pem ubuntu@${{ secrets.EC2_IP }} "
            cd /app &&
            git pull origin main &&
            npm install &&
            pm2 restart app
          "
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **GitHub Actions** | GitHub's native CI/CD platform — pipelines defined as YAML, runs on GitHub's infrastructure |
| **Workflow** | A YAML file in `.github/workflows/` that defines an automated pipeline |
| **Trigger (`on`)** | The event that starts a workflow — push, PR, schedule, manual dispatch |
| **Job** | A set of steps that runs on one runner — jobs run in parallel by default |
| **Step** | A single task within a job — either a `uses` action or a `run` command |
| **Runner** | The virtual machine that executes a job's steps |
| **GitHub-hosted runner** | A managed VM provided by GitHub — discarded after each job |
| **Self-hosted runner** | A machine you manage — registered with GitHub to run jobs |
| **Action** | A reusable, pre-built step from the GitHub Marketplace (e.g. `actions/checkout`) |
| **`uses`** | References a pre-built action by its repo path and version tag |
| **`run`** | Executes a shell command directly on the runner |
| **Secret** | An encrypted value stored in GitHub — injected into workflows at runtime, never logged |
| **`${{ secrets.NAME }}`** | Syntax for referencing a stored secret inside a workflow |
| **Marketplace** | GitHub's library of 20,000+ community and official actions |
| **Minutes quota** | GitHub's monthly limit on CI/CD compute time for private repositories |
| **Platform lock-in** | The cost of migrating away from a tightly coupled tool — workflows must be rewritten |
| **`workflow_dispatch`** | A trigger that allows manual workflow runs from the GitHub UI |
| **Environment secret** | A secret scoped to a specific deployment environment (staging, production) |
| **`needs`** | A job dependency keyword — makes one job wait for another to complete |

---

## 📂 Summary of Tasks
- ✅ Understood: What GitHub Actions is and how it differs from standalone CI/CD tools.
- ✅ Understood: The platform lock-in trade-off — powerful within GitHub, costly to migrate away from.
- ✅ Learned: The `.github/workflows/` structure and core YAML fields.
- ✅ Built: A Python unit test pipeline — checkout, setup, install, test — running on a GitHub-hosted runner.
- ✅ Understood: How GitHub-hosted runners work — provisioned on demand, discarded after each job.
- ✅ Understood: When to use self-hosted runners — specific hardware, internal network access, security policy.
- ✅ Practiced: Secrets management — storing and referencing sensitive values in workflows.
- ✅ Compared: GitHub Actions vs Jenkins — advantages, disadvantages, and the decision framework.

---

## 💡 My Takeaway

The runner model is what makes GitHub Actions fundamentally different from Jenkins. Every job starts on a clean, freshly provisioned VM — no environment drift, no "works on the CI server but not locally" problems, no accumulated state from previous runs. The trade-off is cold start time, but for most pipelines that's negligible.

The platform lock-in point deserves more attention than it usually gets. GitHub Actions YAML is not portable — it only runs on GitHub. A company that builds 3 years of CI/CD workflows in Actions and then decides to move to GitLab or Azure DevOps is looking at a complete rewrite. This is exactly the same argument for using Terraform over CloudFormation. The more you invest in a platform-specific tool, the more expensive the exit becomes. Knowing this doesn't mean avoiding GitHub Actions — it means making the choice with full awareness of the commitment.

---

## 🔗 Resources
* [Abhishek Veeramalla DevOps Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
* [GitHub Actions Documentation](https://docs.github.com/en/actions)
* [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
* [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
* [Self-Hosted Runners Guide](https://docs.github.com/en/actions/hosting-your-own-runners)
* [GitHub Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
* [GitHub Actions vs Jenkins — Comparison](https://www.jenkins.io/blog/2023/06/05/github-actions-alternative/)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*