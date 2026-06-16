
![Progress](https://img.shields.io/badge/Progress-8%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 8 — Git Branching Strategies & Advanced Git Workflows

## 📝 Topic: Branching Models, Merging Strategies, Cherry Pick & Rebase
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)
**Date:** June 12, 2026

---

## 🎯 Learning Objectives
* Understand what a Git branch is and why isolation is the foundation of stable delivery.
* Know all four branching types: Master, Feature, Release, and Hotfix.
* Study how Kubernetes manages 3,300+ contributors using standard branching practices.
* Authenticate with GitHub using both HTTPS and SSH key methods.
* Create and manage branches with `git checkout -b`.
* Understand and apply the three integration methods: Cherry Pick, Merge, and Rebase.
* Resolve merge conflicts confidently.
* Decide when to use `git merge` vs `git rebase` based on team and project context.

---

## 🌿 Part 1 — What is a Branch?

> *"A branch is a separation in the codebase that allows developers to work on new features or breaking changes without affecting the existing, stable functionality delivered to customers."*

### 🚕 The Cab + Bike Example

Imagine you have a working cab booking application serving real customers. You're asked to add a bike booking feature — a major, potentially breaking change.

**Without branching:**
```
main branch → add bike code → bike code breaks cab features → customers affected ❌
```

**With branching:**
```
main branch  →  stays stable, customers unaffected ✅
      ↓
feature/bike-service  →  develop and test bike feature in isolation
      ↓
merge back to main   →  only when fully tested and ready
```

The main branch always represents what's live and stable. All development risk is contained in isolation.

---

## 🌳 Part 2 — The Four Branching Types

### 1️⃣ Master / Main Branch

The **single source of truth** for the project.

```
Rules:
  ✅ Always deployable — every commit here can go to production
  ✅ Always up to date — reflects the latest stable state
  ❌ Never commit directly for large features
  ❌ Never leave it in a broken state
```

All other branches are created from main and eventually merged back into it.

---

### 2️⃣ Feature Branches

Created for a specific new functionality or experiment. Lives in isolation until tested and approved.

```bash
# Create a feature branch from main
git checkout main
git pull origin main
git checkout -b feature/bike-service

# Work on the feature...
git add .
git commit -m "Add bike service API endpoints"
git commit -m "Add bike service database schema"

# When complete — merge back to main
git checkout main
git merge feature/bike-service
git push origin main

# Clean up
git branch -d feature/bike-service
```

**Naming conventions:**
```
feature/user-authentication
feature/payment-gateway
feature/bike-service
experiment/new-search-algorithm
```

---

### 3️⃣ Release Branches

Created when a set of features is **feature-complete** and entering the stabilization phase.

```
main branch         →  development continues for next release
      ↓
release/v2.1.0      →  freeze features, only bug fixes allowed
                       QA testing, documentation, final checks
      ↓
production deploy   →  tag the release, merge back to main
```

**Why this matters:** While QA is testing release/v2.1.0, the main branch continues receiving new feature work for v2.2.0. The two streams don't interfere.

```bash
# Create release branch
git checkout main
git checkout -b release/v2.1.0

# Only bug fixes on this branch
git commit -m "Fix payment edge case in checkout flow"
git commit -m "Update release notes for v2.1.0"

# Tag and deploy
git tag v2.1.0
git push origin release/v2.1.0 --tags

# Merge fixes back to main so main has the bug fixes too
git checkout main
git merge release/v2.1.0
```

---

### 4️⃣ Hotfix Branches

**Short-lived emergency branches** for critical production bugs. Must be merged into both main AND the active release branch.

```
main branch         →  v2.1.0 is live in production
                       Critical bug discovered
      ↓
hotfix/payment-crash → fix the specific bug only
      ↓
merge → main        →  v2.1.1 tagged and deployed
merge → release/v2.1  →  release branch also gets the fix
```

```bash
# Create hotfix from main (the version in production)
git checkout main
git checkout -b hotfix/payment-crash

git add .
git commit -m "Fix null pointer exception in payment processor"

# Merge to main — deploy immediately
git checkout main
git merge hotfix/payment-crash
git tag v2.1.1
git push origin main --tags

# Also merge to active release branch
git checkout release/v2.1.0
git merge hotfix/payment-crash
git push origin release/v2.1.0

# Clean up
git branch -d hotfix/payment-crash
```

---

## 🌐 Part 3 — Real World: Kubernetes Branching

The Kubernetes repository has **3,300+ contributors** and ships a new release every 3 months. They follow exactly these branching standards.

```
kubernetes/kubernetes on GitHub:
  main             →  active development for next release
  release-1.28     →  v1.28.x stabilization and patches
  release-1.27     →  v1.27.x long-term support patches
  release-1.26     →  v1.26.x critical security fixes only
```

Every feature goes through:
1. A feature branch (often in a fork)
2. A Pull Request with code review
3. Automated CI test suite (thousands of tests)
4. Merge to main only after approval
5. Cherry-picked to release branches if it's a critical fix

> **Takeaway:** Study how Docker, Istio, and Jenkins manage their branches too. Real open-source projects are the best textbook for production Git strategy.

---

## 🔐 Part 4 — Authentication: HTTPS vs SSH

Two ways to authenticate with GitHub:

### HTTPS Authentication

```bash
git clone https://github.com/username/repo.git
# Prompts for username + Personal Access Token (PAT)
# PAT replaces passwords for CLI use since 2021
```

**When to use:** Quick one-off clones, CI/CD pipelines, shared machines.

### SSH Key Authentication (Recommended for daily use)

```bash
# Step 1: Generate an SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"
# Creates: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)

# Step 2: Print the public key
cat ~/.ssh/id_ed25519.pub

# Step 3: Add to GitHub
# GitHub → Settings → SSH and GPG Keys → New SSH Key → paste public key

# Step 4: Test the connection
ssh -T git@github.com
# "Hi username! You've successfully authenticated..."

# Step 5: Clone using SSH URL
git clone git@github.com:username/repo.git
```

**SSH vs HTTPS comparison:**

| | HTTPS | SSH |
|---|---|---|
| **Setup** | Instant | One-time key setup |
| **Daily use** | Prompts for token | Passwordless ✅ |
| **Security** | PAT can expire | Key persists |
| **Best for** | CI/CD, one-time use | Developer workstations |

---

## 🔀 Part 5 — Integration Methods: Merge, Rebase & Cherry Pick

### 🍒 Git Cherry Pick — Take Specific Commits

Pick **one specific commit** from any branch and apply it to your current branch.

```
feature branch:   A → B → C → D → E
                                ↑
                       only want commit D

main branch:      X → Y → Z → [D applied here]
```

```bash
# Get the commit hash you want
git log feature/bike-service --oneline
# a3f8c21 Fix bike pricing calculation  ← want this one
# 7b2e104 Add bike UI components
# 9d1a033 Add bike API endpoints

# Cherry pick just that commit onto current branch
git checkout main
git cherry-pick a3f8c21
```

**When to use:** A critical bug fix exists on a feature branch but you don't want to merge the entire feature yet. Or pulling a specific improvement from an experimental branch.

---

### 🔗 Git Merge — Preserve Full History

Combines two branches and creates a **merge commit** that records the history of both.

```
Before merge:
  main:    A → B → C
                    \
  feature:           D → E → F

After git merge:
  main:    A → B → C → G (merge commit)
                    \  /
  feature:           D → E → F
```

```bash
git checkout main
git merge feature/bike-service
# Creates commit G with message "Merge branch 'feature/bike-service'"
```

**The result:** Full history is preserved. The branch topology is visible. You can see exactly when and what was merged.

**When to use:** Public/shared branches, open-source PRs, when history traceability matters (audits, compliance).

---

### 🧹 Git Rebase — Clean Linear History

**Rewrites history** by replaying your branch's commits on top of the target branch.

```
Before rebase:
  main:     A → B → C
                 \
  feature:        D → E → F

After git rebase main (on feature branch):
  main:     A → B → C
                      \
  feature:             D' → E' → F'   ← new commits, same changes
```

```bash
# From the feature branch
git checkout feature/bike-service
git rebase main

# Then fast-forward merge (no merge commit needed)
git checkout main
git merge feature/bike-service   # fast-forward, straight line
```

**The result:** History looks like everything was developed in a straight line. No merge commits. Clean `git log`.

**When to use:** Private/local branches before pushing, keeping project history readable in large teams.

---

### ⚖️ Merge vs Rebase — The Key Decision

| Factor | Git Merge | Git Rebase |
|---|---|---|
| **History** | Preserves exact history + branch topology | Rewrites history — clean linear log |
| **Merge commits** | ✅ Creates one | ❌ None |
| **Safety** | Safe on shared branches | ⚠️ Never rebase pushed/shared branches |
| **Readability** | Can get messy with many branches | Clean, linear, easy to follow |
| **Blame/audit** | Full context preserved | Cleaner but rewritten |
| **Best for** | Main branch integration, PRs, open source | Local cleanup before PR, personal branches |

> ⚠️ **The Golden Rule of Rebase:** Never rebase a branch that other developers are working on. Rewriting history on a shared branch means everyone else's local copies are now out of sync — a painful conflict situation.

---

## ⚔️ Part 6 — Conflict Resolution

Conflicts happen when two branches modify the **same lines in the same file**.

```bash
# Attempting merge
git merge feature/bike-service

# Git reports:
# CONFLICT (content): Merge conflict in app.py
# Automatic merge failed; fix conflicts and then commit the result.
```

**Inside the conflicted file:**

```python
<<<<<<< HEAD (your current branch - main)
def get_service_price(service):
    prices = {"cab": 50, "auto": 30}
    return prices.get(service, 0)
=======
def get_service_price(service):
    prices = {"cab": 50, "auto": 30, "bike": 15}
    return prices.get(service, 0)
>>>>>>> feature/bike-service
```

**Resolution steps:**

```bash
# Step 1: Open the file — choose the correct version or combine both
def get_service_price(service):
    prices = {"cab": 50, "auto": 30, "bike": 15}  # kept bike addition
    return prices.get(service, 0)

# Step 2: Stage the resolved file
git add app.py

# Step 3: Complete the merge
git commit -m "Merge feature/bike-service — resolve pricing conflict"
```

**Best practice:** Talk to the other developer before resolving. You might need their context to understand which version is correct. Conflict resolution is a collaboration, not a solo decision.

---

## 📋 Part 7 — The Complete Daily Git Workflow

```bash
# Start of day — sync with remote
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/add-bike-service

# Develop...
git add .
git commit -m "Add bike service data model"
git add .
git commit -m "Add bike service REST endpoints"
git add .
git commit -m "Add unit tests for bike service"

# Keep branch up to date with main during development
git fetch origin
git rebase origin/main    # rebase onto latest main

# Push to remote
git push origin feature/add-bike-service

# Open a Pull Request on GitHub
# → request code review
# → CI tests run automatically
# → reviewers approve

# After approval — merge to main
git checkout main
git merge feature/add-bike-service
git push origin main

# Tag a release
git tag v2.2.0
git push origin v2.2.0

# Clean up
git branch -d feature/add-bike-service
git push origin --delete feature/add-bike-service
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Branch** | An isolated line of development — a pointer to a specific commit |
| **Master / Main** | The primary branch — always stable, always deployable |
| **Feature Branch** | Isolated branch for developing a single new feature |
| **Release Branch** | Stabilization branch for a specific version going to production |
| **Hotfix Branch** | Emergency branch for critical production bug fixes |
| **`git checkout -b`** | Create and switch to a new branch in one command |
| **`git merge`** | Combines two branches — creates a merge commit, preserves history |
| **`git rebase`** | Replays commits on top of another branch — rewrites history, linear log |
| **`git cherry-pick`** | Applies a specific commit from one branch to another |
| **Merge Commit** | A commit with two parents that records where two branches joined |
| **Fast-Forward Merge** | A merge where main simply advances its pointer — no merge commit needed |
| **Merge Conflict** | Two branches modifying the same lines — Git can't auto-resolve |
| **Conflict Markers** | `<<<<<<<`, `=======`, `>>>>>>>` — Git's way of marking conflicting changes |
| **`git tag`** | A permanent label on a specific commit — used for version releases |
| **HTTPS auth** | GitHub authentication via username + Personal Access Token |
| **SSH auth** | GitHub authentication via public/private key pair — passwordless |
| **`ed25519`** | Modern, fast, secure SSH key algorithm — preferred over RSA |
| **Pull Request (PR)** | A request to merge a branch — includes code review and CI on GitHub |
| **`git fetch`** | Downloads remote changes without merging them locally |
| **`git pull`** | `git fetch` + `git merge` — downloads and applies remote changes |
| **Fork** | A personal copy of someone else's repository on GitHub |
| **Golden Rule of Rebase** | Never rebase a branch that others are working on — breaks shared history |

---

## 📂 Summary of Tasks
- ✅ Understood: What a branch is and why isolation protects production stability.
- ✅ Learned: All four branch types — Master, Feature, Release, Hotfix — with real examples.
- ✅ Studied: Kubernetes as a real-world case study for branching at 3,300+ contributors.
- ✅ Set up: SSH key authentication with GitHub — `ssh-keygen`, copy public key, test connection.
- ✅ Practiced: `git checkout -b` to create and switch to feature branches.
- ✅ Understood: `git cherry-pick` — selecting specific commits across branches.
- ✅ Understood: `git merge` — combining branches with full history preserved.
- ✅ Understood: `git rebase` — rewriting history for a clean linear log.
- ✅ Understood: When to use merge vs rebase — the golden rule of rebase.
- ✅ Practiced: Resolving a merge conflict — conflict markers, resolution, commit.

---

## 💡 My Takeaway

The merge vs rebase distinction was the most practically useful thing today. Both combine branches but with fundamentally different philosophies: merge says "show me exactly what happened and when," rebase says "show me a clean story of what was built." Neither is wrong — the choice depends on whether you're on a shared branch (always merge) or a local branch before a PR (rebase is fine, and often preferred).

The Kubernetes example made branching strategy feel real rather than theoretical. 3,300 contributors, releases every 3 months, no chaos — because everyone follows the same four-branch model. Good branching isn't bureaucracy, it's the thing that makes scale possible.

The hotfix merge-to-both requirement is also worth internalizing. A hotfix that only goes to main but not to the active release branch means the next patch release ships with the same bug you just fixed. That's a trap that's easy to miss under pressure.

---

## 🔗 Resources
* [Abhishek Veermalla Date Playlist](https://youtube.com/playlist?list=PLdpzxOOAlwvIc1TjTwopNSjRJkzES2ZXk&si=kx2Ia6UsQ1Oou6Vg)
* [Kubernetes GitHub — Real branching in action](https://github.com/kubernetes/kubernetes/branches)
* [Atlassian Git Branching Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)
* [Git Cherry Pick Docs](https://git-scm.com/docs/git-cherry-pick)
* [Git Rebase Docs](https://git-scm.com/docs/git-rebase)
* [GitHub SSH Key Setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
* [Pro Git Book — Branching](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
* [Git Flight Rules — conflict resolution guide](https://github.com/k88hudson/git-flight-rules)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
