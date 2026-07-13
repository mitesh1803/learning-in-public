![Progress](https://img.shields.io/badge/Progress-34%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 11 — CloudFormation Templates (CFT)

## 📝 Topic: Infrastructure as Code on AWS — Template Structure, Stacks, Drift Detection & CFT vs. Terraform
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 12, 2026
---

## 🎯 Learning Objectives
* Understand Infrastructure as Code (IaC) and where CloudFormation fits as AWS's native implementation.
* Understand CFT's declarative model and why that matters for provisioning.
* Distinguish AWS CLI's use case from CFT's use case.
* Learn every block in a CFT template's anatomy, and which one is actually mandatory.
* Understand Stacks as the management unit for a template's resources.
* Understand Drift Detection and why it matters operationally.
* Learn practical tooling (Designer, VS Code extensions) for writing templates efficiently.
* Build a strategic answer for the classic "CFT vs. Terraform" interview question.

---

## 📜 Part 1 — Infrastructure as Code (IaC) & AWS CFT

### What Is IaC?

```
Infrastructure as Code = managing, provisioning, and updating
cloud infrastructure by writing CODE
                        instead of
manual configuration via a web console
```

### Where CFT Fits

```
AWS CFT (CloudFormation Templates) = AWS's NATIVE tool
                                      for implementing IaC

  You write human-readable YAML/JSON
        ↓
  CloudFormation service translates it into
  specific AWS API calls
        ↓
  Infrastructure gets built exactly as described
```

> CFT acts as a middleman between your declared intent and the raw AWS API calls — the same abstraction-layer role the AWS CLI plays (from AWS Day 10), just applied to entire multi-resource architectures instead of individual ad hoc commands.

### The Declarative Model

```
Declarative ("what you see is what you have"):
  → You describe the END STATE you want
  → AWS figures out the provisioning SEQUENCE itself

  Contrast with imperative ("how"):
    Step 1: create VPC
    Step 2: create subnet
    Step 3: create route table
    ...

  Declarative equivalent:
    "I want: 1 VPC, 2 subnets, 1 route table, connected like this"
    → CloudFormation determines the correct order of operations
```

### Version Control

```
Because templates are just TEXT FILES:
  → Store them in Git repositories
  → Or save them in an S3 bucket

  → Enables auditing, code review, and tracking
    infrastructure changes over time — the same way
    application code changes are tracked
```

---

## ⚖️ Part 2 — AWS CLI vs. AWS CFT

```
Same code-based interface, ENTIRELY different use cases:
```

| Tool | Best for | Nature |
|---|---|---|
| **AWS CLI** | Short, quick, ad hoc queries/actions (e.g., listing S3 buckets) | NOT declarative — one-off commands |
| **AWS CFT** | Long-term, multi-resource architectures (VPC + route tables + ALB + EC2, all together) | Declarative IaC |

> **The distinction, sharpened:** the CLI is for asking AWS a question or performing a single action right now. CFT is for describing an entire architecture's desired state, once, and having AWS build (and later reconcile) it as a single managed unit.

---

## 🧱 Part 3 — Structure and Components of a CFT

### Format Choice: YAML or JSON

```
Both are supported — but YAML is recommended:

  → Supports line commenting (#) — critical for team collaboration
  → Cleaner, Python-like indentation vs. JSON's messy curly braces
  → Generally less complex to maintain over time
```

### The Anatomy of a Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: "Launches a single EC2 instance for testing"

Metadata:
  Author: "Mitesh"

Parameters:
  InstanceTypeParam:
    Type: String
    Default: t2.micro

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0abcdef1234567890

Conditions:
  IsProd: !Equals [!Ref EnvType, "prod"]

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceTypeParam
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]

Outputs:
  InstancePublicIP:
    Value: !GetAtt MyEC2Instance.PublicIp
```

| Component | Mandatory? | Purpose |
|---|---|---|
| **`AWSTemplateFormatVersion`** | No | The CFT spec version (typically `2010-09-09`) |
| **`Description`** | No | Explains what the template creates — important for code review |
| **`Metadata`** | No | Administrative info: author, owner, project name |
| **`Parameters`** | No | Runtime inputs — dynamically pass values like instance size or AMI ID |
| **`Rules`** | No | Validates parameter inputs, blocking unsafe configurations |
| **`Mappings`** | No | A lookup table mapping variables to environments/regions |
| **`Conditions`** | No | Controls resource creation based on criteria (e.g., Dev vs. Prod) |
| **`Resources`** | **✅ Yes** | Defines the actual AWS components to create |
| **`Outputs`** | No | Returns details post-execution (e.g., a new instance's public IP) |

> **The one thing to remember:** `Resources` is the ONLY mandatory block. Every other block exists to make templates more dynamic, reusable, and self-documenting — but a minimal valid template needs nothing more than a `Resources` section.

---

## 📦 Part 4 — CloudFormation Stacks & Drift Detection

### Stacks — The Management Unit

```
When a template is submitted (via Console or CLI):
  → It becomes a STACK
  → CloudFormation builds, tracks, and treats EVERY resource
    defined in that template as a SINGLE ENTITY

  → Update the stack → CFT calculates and applies the diff
  → Delete the stack → CFT tears down every resource it created,
    in the correct dependency order
```

### Drift Detection

```
Problem it solves:
  An engineer bypasses IaC discipline and manually changes
  a resource directly in the Console
  (e.g., turning OFF versioning on an S3 bucket
   that the template says should have versioning ON)

Solution: click "Detect Drift"
  → CFT compares the LIVE infrastructure state
    against the ORIGINAL template definition
  → Reports the exact variance found
  → Labels the affected resource as "Drifted"
```

> **Why this matters operationally:** without Drift Detection, a manually-changed resource silently diverges from what the template claims is true — the next `stack update` might not even notice, or worse, might behave unexpectedly because CFT's internal state no longer matches reality. Drift Detection surfaces that gap explicitly instead of letting it accumulate invisibly.

---

## 🛠️ Part 5 — Writing Efficient Templates: Tips & Tools

```
Writing CFT files entirely from memory isn't realistic —
the number of possible properties across every AWS
service is enormous. Use these instead:

1. AWS CloudFormation Designer
   → Drag-and-drop visual tool inside the Console
   → Generates valid JSON/YAML in real time as you
     visually place infrastructure elements

2. AWS Official Documentation ("Template Reference")
   → Comprehensive reference for copy-pasting correct
     structural syntax directly into a project

3. VS Code Extensions
   → YAML (by Red Hat): catches syntax/indentation errors
   → AWS Toolkit: connects the IDE to live AWS config,
     provides auto-complete for resource blocks
```

> **Practical workflow this suggests:** don't memorize every property key — look them up via the Template Reference or lean on the AWS Toolkit's autocomplete, and let the YAML extension catch indentation mistakes before they become a failed stack deployment.

---

## 🆚 Part 6 — Interview Preparation: CFT vs. Terraform

```
Classic interview question: "Why would you use Terraform
                              instead of AWS CFT, or vice versa?"

Strategic framing:

  Multi-Cloud / Hybrid Strategy → TERRAFORM
    → Supports AWS, Azure, GCP, on-premise, all in one tool
    → Organizations NOT fully committed to AWS alone
      will heavily favor Terraform

  Single-Cloud (AWS-only) Specialization → CFT
    → Natively integrates with AWS APIs, no external
      plugins/providers needed
    → For an organization with no roadmap to expand
      beyond AWS, CFT offers a seamless, native,
      fully-supported automation lifecycle
```

> **How to actually use this in an interview:** don't just recite "Terraform is multi-cloud" — tie the answer to the ORGANIZATION'S actual strategy. If they're all-in on AWS with zero multi-cloud ambition, CFT is a perfectly defensible (arguably better-integrated) choice; the "Terraform is always better" answer misses that nuance.

---

## 🏋️ Part 7 — Day 11 Assignment

```
Goal: Build an EC2 Instance STRICTLY using a CloudFormation template
(no manual Console clicking to create the instance itself)

Steps:
  1. Use the AWS Toolkit (VS Code) or the Template Reference docs
     to identify the required Resource keys:
       ImageId (AMI), InstanceType, KeyName
  2. Write the template (YAML preferred)
  3. Upload it as a NEW STACK in a test AWS account
  4. Verify the instance boots successfully
  5. Turn ON Drift Detection and manually change something
     in the Console afterward, to confirm tracking works
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **IaC (Infrastructure as Code)** | Managing infrastructure via code/text files instead of manual console configuration |
| **AWS CFT** | AWS's native CloudFormation Templates service for implementing IaC |
| **Declarative Model** | Describing the desired end-state; the tool determines the provisioning sequence |
| **Stack** | The management unit in CloudFormation — a group of resources tracked and treated as one entity |
| **Drift Detection** | A feature comparing live infrastructure against the original template to surface manual out-of-band changes |
| **`Resources`** | The one mandatory block in a CFT template, defining the actual AWS components to create |
| **`Parameters`** | Runtime inputs allowing dynamic values (e.g., instance size, AMI) to be passed into a template |
| **`Mappings`** | A lookup table mapping variables to specific environments or regions |
| **`Outputs`** | Post-execution return values (e.g., a created instance's public IP) |
| **CloudFormation Designer** | A drag-and-drop visual tool generating valid CFT code in real time |
| **AWS Toolkit (VS Code)** | An extension connecting the IDE to live AWS config with auto-complete for CFT resources |

---

## 📂 Summary of Tasks
- ✅ Understood: Infrastructure as Code and CloudFormation's role as AWS's native IaC tool.
- ✅ Understood: CFT's declarative model — describing end-state rather than step-by-step instructions.
- ✅ Distinguished: AWS CLI (quick, ad hoc, non-declarative) vs. CFT (long-term, declarative, multi-resource).
- ✅ Learned: Every block in a CFT template's anatomy and confirmed `Resources` is the only mandatory one.
- ✅ Understood: Why YAML is generally preferred over JSON for CFT templates.
- ✅ Understood: Stacks as CloudFormation's management unit for tracking a template's resources as one entity.
- ✅ Understood: Drift Detection and how it surfaces manual, out-of-band Console changes.
- ✅ Noted: CloudFormation Designer, AWS Template Reference, and VS Code extensions as practical writing aids.
- ✅ Built: A strategic, organization-aware answer for the "CFT vs. Terraform" interview question.
- ✅ Assigned: Build and deploy an EC2 instance strictly via a CFT template, with Drift Detection enabled.

---

## 💡 My Takeaway

Drift Detection is the feature I most want to actually use going forward, not just know about — it directly names a failure mode I've definitely been guilty of during earlier hands-on sessions in this series (making a quick manual tweak in the Console "just to test something" and never reconciling it back into the template). Having a built-in mechanism to surface exactly that gap, rather than relying on memory or discipline alone, is a genuinely practical safety net for real IaC workflows.

The "`Resources` is the only mandatory block" detail is a good one to hold onto for both practical use and interviews — it reframes the other nine blocks (Parameters, Mappings, Conditions, Outputs, etc.) as progressively more sophisticated tooling layered on top of a genuinely minimal core, rather than all being equally essential from day one.

The CFT-vs-Terraform framing finally gave me language for something I'd sensed but hadn't articulated well: the "right" IaC tool isn't universal, it's a direct function of an organization's cloud strategy. Given my own portfolio projects (Intervio on AWS EC2, GrowEasy, etc.) are currently single-cloud AWS, CFT is a genuinely reasonable choice to practice alongside Terraform — not just a fallback for when Terraform "isn't available."

---

## 📈 Next Up
**AWS Day 12:** AWS CI/CD ecosystem — CodeCommit, CodePipeline, CodeBuild, and CodeDeploy, starting with setting up a private CodeCommit repository.

---

## 🔗 Resources
* [AWS CloudFormation Documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
* [CloudFormation Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-reference.html)
* [CloudFormation Drift Detection](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-drift.html)
* [AWS Toolkit for VS Code](https://aws.amazon.com/visualstudiocode/)
* [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*