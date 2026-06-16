![Progress](https://img.shields.io/badge/Progress-2%25-orange?style=for-the-badge&logo=progress)

# 🚀 Day 02: SDLC & Where DevOps Fits In

## 📝 Topic: Software Development Life Cycle & DevOps Integration
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)  
**Date:** June 7, 2026

---

## 🎯 Learning Objectives
* Understand what SDLC is and why the entire IT industry follows it.
* Map out all 6 phases of a standard SDLC cycle.
* Identify exactly where DevOps plugs into the SDLC and why.

---

## 🏗️ What is SDLC?

> **"SDLC is a set of standardized phases used across the IT industry to ensure the creation of high-quality software."**

It applies universally — from early-stage startups to large MNCs. Rather than every team inventing their own process, SDLC gives everyone a shared, structured approach to designing, developing, and testing products.

---

## 🔄 The 6 Phases of SDLC

SDLC is a **circular process** — once a feature ships, the cycle restarts for the next one.

```
Planning → Defining → Designing → Building → Testing → Deployment
    ↑                                                        |
    └────────────────────────────────────────────────────────┘
```

### 📋 Phase 1: Planning & Requirements
Gather input from stakeholders and customers. Assess the feasibility of what needs to be built before any work begins.

### 📄 Phase 2: Defining
Formally document all requirements. The primary output is the **SRS (Software Requirement Specification)** — the source of truth for everyone involved.

### 🎨 Phase 3: Designing
Two layers of design are produced:
* **HLD (High-Level Design)** — overall system architecture and how components connect.
* **LLD (Low-Level Design)** — module-specific logic and implementation details.

### 💻 Phase 4: Building (Development)
Developers write code and push it to a version control repository like **Git**. This is where the actual product takes shape.

### 🧪 Phase 5: Testing
The **QA (Quality Assurance)** team validates the application — catching bugs, edge cases, and regressions before anything reaches real users.

### 🚢 Phase 6: Deployment
The tested build is promoted to the **production environment**, making it live and accessible to customers.

---

## ⚙️ How DevOps Improves the SDLC

DevOps engineers can participate in any phase, but their **primary focus is automating three stages: Building, Testing, and Deployment**.

| Phase | Without DevOps | With DevOps |
|---|---|---|
| **Building** | Manual code handoffs between teams | Automated pipelines trigger on every commit |
| **Testing** | QA runs tests manually after dev completes | Tests run automatically on every build |
| **Deployment** | Manual steps, high risk of human error | Automated, repeatable, consistent releases |

### 🧱 The 3 DevOps Wins
1. **Efficiency 🤖:** Automation scripts eliminate manual bottlenecks at every stage.
2. **Speed 🚀:** Code moves from a developer's workstation to production in a fraction of the time.
3. **Culture 🤝:** DevOps is a bridge — it fosters collaboration between Dev and Ops teams to deliver high-quality code quickly and reliably.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **SDLC** | Software Development Life Cycle — standardized phases for building software |
| **SRS** | Software Requirement Specification — formal document capturing what needs to be built |
| **HLD** | High-Level Design — system-wide architecture decisions |
| **LLD** | Low-Level Design — module-level logic and implementation details |
| **QA** | Quality Assurance — the team responsible for testing and validation |
| **Production** | The live environment where real users access the software |
| **CI/CD** | The DevOps practice of automating Build → Test → Deploy continuously |

---

## 📂 Summary of Tasks
- [x] Watched: Day 2 - SDLC & DevOps Integration.
- [x] Understood: All 6 phases of SDLC and their outputs.
- [x] Understood: The 3 stages where DevOps automation has the highest impact.

---

## 💡 My Takeaway

SDLC is not just corporate process overhead — it's what separates *"we shipped something"* from *"we shipped something reliably."* What stood out most is how DevOps plugs into exactly the three stages where things break down without automation: building, testing, and deployment. Those are the stages with the most manual handoffs and the highest risk of human error. That's not a coincidence — that's by design.

---

## 📈 Next Up
**Day 03:** Hands-on with Linux fundamentals — the foundation of every DevOps tool in the stack.

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*