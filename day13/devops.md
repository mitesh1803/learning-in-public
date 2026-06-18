![Progress](https://img.shields.io/badge/Progress-13%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 13 — CI/CD Fundamentals: From Concept to Modern Pipelines

## 📝 Topic: Continuous Integration & Continuous Delivery — Theory, Pipeline Stages & Tool Landscape
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** June 18, 2026

---

## 🎯 Learning Objectives
* Define CI and CD precisely — and understand where one ends and the other begins.
* Map out all standard pipeline stages: Unit Test → Static Analysis → Vulnerability → E2E → Deploy.
* Understand why CI/CD exists — the cost of manual delivery at scale.
* Contrast Legacy CI/CD (Jenkins) with Modern event-driven CI/CD (GitHub Actions).
* Understand the "zero compute wastage" model of container-based pipelines.
* Know the key CI/CD tools and when each is relevant.

---

## 🔄 Part 1 — What is CI/CD?

### The Core Problem CI/CD Solves

```
Without CI/CD:
  Developer finishes feature
        ↓
  Manually runs tests (or skips them)
        ↓
  Manually builds the application
        ↓
  Manually deploys to staging
        ↓
  QA manually tests
        ↓
  Manually deploys to production
        ↓
  Hopes nothing breaks

Time from code to customer: days to weeks
Human error: high
Consistency: none
```

```
With CI/CD:
  Developer pushes code to GitHub
        ↓
  Pipeline triggers automatically
        ↓
  All tests run, all checks pass
        ↓
  Build artifact created
        ↓
  Deployed to staging automatically
        ↓
  Deployed to production (on approval or fully automatic)

Time from code to customer: minutes to hours
Human error: eliminated from repetitive steps
Consistency: identical every time
```

---

### 🔵 Continuous Integration (CI)

> *"The practice of integrating code changes frequently and verifying each integration with automated tests."*

**What CI does:**
* Detects when code is pushed or a PR is opened
* Automatically runs the test suite
* Reports whether the new code passes or breaks existing functionality
* Blocks merging if tests fail

**The key word is "continuous"** — not once a week, not at release time. Every single commit triggers the pipeline. Bugs are caught within minutes of being introduced, by the same developer who wrote them, while the context is still fresh.

---

### 🟢 Continuous Delivery (CD)

> *"The practice of automatically deploying code that has passed CI to staging or production environments."*

**What CD does:**
* Takes the build artifact that passed CI
* Deploys it automatically to target environments
* Ensures the deployment process is repeatable and auditable

**CI/CD together:**

```
Code Push → CI (test, build, verify) → CD (deploy)
                                            ↓
                                    Staging (auto)
                                            ↓
                                    Production (auto or approval gate)
```

---

## 🏗️ Part 2 — The Standard Pipeline Stages

Every production CI/CD pipeline includes these stages. Skipping any one of them is a risk.

### Stage 1 — Unit Testing ✅

Tests individual functions or components in isolation.

```python
# Calculator app — the function being tested
def add(a, b):
    return a + b

# Unit test
def test_add():
    assert add(2, 3) == 5       # passes
    assert add(-1, 1) == 0      # passes
    assert add(0, 0) == 0       # passes
```

**What it catches:** Logic bugs introduced by the developer — wrong formula, off-by-one errors, null handling failures.

**Characteristics:**
* Fast — run in milliseconds
* No external dependencies (mock databases, APIs)
* The first and cheapest line of defence

---

### Stage 2 — Static Code Analysis 🔍

Checks code quality without executing it.

```bash
# Python — flake8 checks formatting, style, complexity
flake8 src/

# JavaScript — ESLint
eslint src/

# SonarQube — deeper analysis: code smells, duplication, cognitive complexity
```

**What it catches:**
* Formatting violations (inconsistent indentation, line length)
* Unused variables, unreachable code
* Memory inefficiency patterns
* High cyclomatic complexity (code that's hard to maintain and test)

**Why it matters:** Code that passes tests can still be unmaintainable, inconsistent, or wasteful. Static analysis enforces team-wide standards automatically.

---

### Stage 3 — Vulnerability Testing 🔒

Scans code and dependencies for known security flaws.

```bash
# Scan Python dependencies for CVEs
pip-audit

# JavaScript dependency scan
npm audit

# Container image scanning
trivy image my-app:latest

# SAST (Static Application Security Testing)
bandit -r src/    # Python security linter
```

**What it catches:**
* Dependencies with known CVEs (Common Vulnerabilities and Exposures)
* Hardcoded secrets (API keys, passwords accidentally committed)
* Insecure coding patterns (SQL injection risks, unsafe deserialization)

**Why it must be automated:** New CVEs are published daily. A dependency that was safe last month may be vulnerable today. Manual tracking is impossible at scale.

---

### Stage 4 — Automation / Functional Testing 🤖

End-to-end tests that verify the entire application works as users experience it.

```
Unit tests:    test the function in isolation
Integration:   test that services work together
E2E tests:     test the entire user journey

E2E example:
  1. Open the app URL
  2. Log in with test credentials
  3. Add item to cart
  4. Complete checkout
  5. Verify confirmation email was sent
  6. Verify inventory was decremented
```

**What it catches:** Integration regressions — cases where individual components pass unit tests but break when combined. The "it works on my machine" class of bugs.

---

### Stage 5 — Reporting 📊

Captures and communicates the health of every build.

```
Build #147 — PASSED ✅
  Unit Tests:          423/423 passed  (12.3s)
  Static Analysis:     0 violations
  Vulnerability Scan:  0 critical CVEs
  E2E Tests:           87/87 passed   (4m 32s)
  Coverage:            84.2%

Build #148 — FAILED ❌
  Unit Tests:          421/423 passed
  FAILED: test_payment_processor_timeout
  FAILED: test_refund_calculation_edge_case
```

Reports are sent to Slack, email, or GitHub PR comments. The team knows immediately what broke and who introduced it.

---

### Stage 6 — Deployment 🚢

The final stage — moving the verified artifact to the target environment.

```
Deploy targets:
  Kubernetes:  kubectl apply -f deployment.yaml
  EC2:         ssh + pull new image + restart service
  ECS/EKS:     update service with new task definition
  Lambda:      aws lambda update-function-code
```

```
Environments in sequence:
  dev (automatic) → staging (automatic) → production (approval gate or automatic)
```

---

### 📋 Pipeline at a Glance

```
Code Push (GitHub)
       ↓
┌─────────────────────────────────────────────────────────┐
│                   CI/CD Pipeline                        │
│                                                         │
│  1. Unit Tests      → fast feedback, logic bugs         │
│  2. Static Analysis → code quality, formatting          │
│  3. Vulnerability   → CVEs, secrets, insecure patterns  │
│  4. E2E Tests       → full user journey, regressions    │
│  5. Reporting       → build health, coverage, trends    │
│  6. Deploy          → staging → production              │
│                                                         │
└─────────────────────────────────────────────────────────┘
       ↓
Customer receives working, tested, secure code
```

---

## 🏛️ Part 3 — Legacy CI/CD: Jenkins

Jenkins was the original CI/CD standard and remains widely used. Understanding it is essential — most large enterprises still run Jenkins.

### How Jenkins Works

```
Jenkins Architecture:
  Jenkins Master (controller)
       ↓ distributes jobs to
  Jenkins Agents (workers)
  Agent 1 | Agent 2 | Agent 3 | ... Agent N
```

**Jenkinsfile — pipeline as code:**

```groovy
pipeline {
    agent any

    stages {
        stage('Unit Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Static Analysis') {
            steps {
                sh 'eslint src/'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-app:${BUILD_NUMBER} .'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh 'kubectl apply -f k8s/staging/'
            }
        }
    }

    post {
        failure {
            mail to: 'team@company.com',
                 subject: "Build ${BUILD_NUMBER} failed",
                 body: "Check Jenkins for details."
        }
    }
}
```

### ⚠️ The Legacy Problem

```
Jenkins setup at scale:
  Master node:        Always running      → $150/month
  Agent 1:            Always running      → $100/month
  Agent 2:            Always running      → $100/month
  Agent 3:            Always running      → $100/month
  Agent 4:            Always running      → $100/month

Total:                                      $550/month
Utilization:  Agents sit idle most of the day

Scale to 100 agents for a large org:      $10,000+/month
Most of that compute is idle.
```

**The three Jenkins pain points:**
* **Cost** — dedicated machines running 24/7 regardless of pipeline activity
* **Maintenance** — Jenkins itself, plugins, JVM, agent dependencies all need patching
* **Scaling** — manually provisioning more agents when load spikes; idle capacity during quiet periods

---

## ⚡ Part 4 — Modern CI/CD: Event-Driven + Container-Based

### The Fundamental Shift

```
Legacy model:
  Infrastructure always exists, waiting for pipelines to use it

Modern model:
  Infrastructure is created ON DEMAND when a pipeline runs
  Destroyed immediately after the pipeline finishes
  Zero cost when no pipelines are running
```

### How GitHub Actions Works

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest    # a fresh container, spun up on demand

    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Run unit tests
        run: npm test

      - name: Run static analysis
        run: npm run lint

      - name: Vulnerability scan
        run: npm audit --audit-level=high
```

**What happens when code is pushed:**

```
1. GitHub detects the push
2. Spins up a fresh Ubuntu container in GitHub's infrastructure
3. Runs all steps in sequence
4. Reports results as PR checks
5. Container is destroyed — no ongoing cost
```

### Zero Compute Wastage — The Kubernetes Example

The Kubernetes project has thousands of contributors and hundreds of PRs daily. Every PR triggers a full test suite.

```
Old model:  keep 500 Jenkins agents running 24/7 to handle peak load
            → most are idle most of the time
            → enormous cost

New model:  GitHub Actions spins up exactly as many containers as needed
            → 500 PRs open simultaneously? 500 containers launch
            → all PRs close? 0 containers running
            → pay only for actual compute used
```

---

## 🛠️ Part 5 — CI/CD Tool Landscape

| Tool | Type | Best For | Key Characteristic |
|---|---|---|---|
| **Jenkins** | Self-hosted | Large enterprises with existing Jenkins, complex custom pipelines | Most flexible, highest maintenance |
| **GitHub Actions** | Cloud-native (SaaS) | Projects already on GitHub, modern event-driven pipelines | Zero infrastructure, tight GitHub integration |
| **GitLab CI/CD** | Cloud + self-hosted | Teams using GitLab for source control | Native to GitLab, strong Docker/K8s integration |
| **Travis CI** | SaaS | Open source projects (historically free for OSS) | Simple YAML config, good OSS history |
| **CircleCI** | SaaS | Fast pipelines, parallelism | Performance-focused, strong caching |

### When to Use Which

```
Starting fresh on GitHub?          → GitHub Actions (zero setup)
Using GitLab?                      → GitLab CI/CD (built in)
Large enterprise with 5-year       
Jenkins investment?                → Jenkins (or migrate gradually)
Need maximum pipeline speed?       → CircleCI (parallelism, caching)
Open source project?               → GitHub Actions or Travis CI
```

---

## 🔑 Part 6 — CI/CD in the SDLC Context

Recall from Day 03 — SDLC: DevOps owns the Build → Test → Deploy stages. CI/CD is the automation engine that makes those three stages happen without human intervention.

```
SDLC Phase      →   CI/CD Automation
─────────────────────────────────────────
Build           →   Compile, package, containerize (CodeBuild, GitHub Actions)
Test            →   Unit, static, vulnerability, E2E (all automated in pipeline)
Deploy          →   Push to staging/production (CodeDeploy, Argo CD, Helm)
Monitor         →   Observe pipeline metrics, alert on failures (CloudWatch, Datadog)
```

> **The CI/CD pipeline IS the automated SDLC for every code change.**

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **CI** | Continuous Integration — automated testing on every code commit |
| **CD** | Continuous Delivery/Deployment — automated deployment of tested code |
| **Pipeline** | An ordered sequence of automated stages from code to deployment |
| **Unit Test** | Tests a single function/component in isolation |
| **Static Code Analysis** | Examining code without executing it — checks style, complexity, patterns |
| **Vulnerability Scan** | Checking for known CVEs in dependencies and insecure code patterns |
| **CVE** | Common Vulnerability and Exposure — a publicly known security flaw |
| **E2E Test** | End-to-End test — simulates a full user journey through the application |
| **Regression** | A bug introduced by new code that breaks previously working functionality |
| **Build Artifact** | The output of the build stage — a Docker image, JAR, or binary |
| **Approval Gate** | A manual confirmation step before deploying to production |
| **Jenkins** | Open-source automation server — the legacy CI/CD standard |
| **Jenkinsfile** | A pipeline definition written in Groovy DSL, stored in the repo |
| **GitHub Actions** | GitHub's native CI/CD — event-driven, container-based, zero infrastructure |
| **Workflow** | A GitHub Actions pipeline definition (`.github/workflows/*.yml`) |
| **Runner** | The compute environment where a CI/CD job executes |
| **Event-Driven** | Pipeline only runs when triggered by an event (push, PR, merge) — no idle compute |
| **Zero Compute Wastage** | Infrastructure exists only while a pipeline is running — no always-on agents |
| **Idempotent Pipeline** | Running the same pipeline on the same code always produces the same result |

---

## 📂 Summary of Tasks
- ✅ Defined: Continuous Integration — test on every commit.
- ✅ Defined: Continuous Delivery — deploy automatically after CI passes.
- ✅ Mapped: All 6 pipeline stages — Unit Test → Static Analysis → Vulnerability → E2E → Report → Deploy.
- ✅ Understood: Why each stage exists and what class of bugs it catches.
- ✅ Understood: Jenkins architecture (Master + Agents) and its cost/maintenance limitations at scale.
- ✅ Written: A sample Jenkinsfile pipeline and a GitHub Actions workflow YAML.
- ✅ Understood: The event-driven model — containers on demand, zero idle cost.
- ✅ Compared: Jenkins vs GitHub Actions vs GitLab CI/CD vs Travis CI vs CircleCI.
- ✅ Connected: CI/CD to the SDLC phases covered in Day 03.

---

## 💡 My Takeaway

The "zero compute wastage" concept reframed how I think about infrastructure cost for CI. With Jenkins, you're essentially paying for servers to sit and wait. With GitHub Actions, you're paying only for the seconds your pipeline actually runs. For a project like Kubernetes with thousands of contributors, that difference is enormous — you'd need massive infrastructure to handle peak PR load with Jenkins, but GitHub Actions scales to exactly what's needed, exactly when it's needed, and drops to zero otherwise.

The pipeline stage breakdown also changed how I think about testing. It's not "tests" as one undifferentiated category — it's four different types catching four different categories of problems: unit tests catch logic bugs (seconds to run), static analysis catches code quality issues (seconds), vulnerability scans catch security risks (seconds to minutes), and E2E tests catch integration regressions (minutes). They're ordered from fastest to slowest deliberately — fail fast on the cheap checks before running the expensive ones.

---

## 📈 Next Up
**Day 19:** Jenkins hands-on — installing Jenkins, writing a Jenkinsfile for the NodeJS app from Day 12, and running a real pipeline with all 6 stages.

---

## 🔗 Resources
* [GitHub Actions Documentation](https://docs.github.com/en/actions)
* [Jenkins Documentation](https://www.jenkins.io/doc/)
* [GitLab CI/CD Docs](https://docs.gitlab.com/ee/ci/)
* [OWASP — Vulnerability Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
* [Martin Fowler — Continuous Integration](https://martinfowler.com/articles/continuousIntegration.html)
* [GitHub Actions — Starter Workflows](https://github.com/actions/starter-workflows)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
