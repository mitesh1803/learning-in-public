![Progress](https://img.shields.io/badge/Progress-36%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 17 — AWS Lambda & Serverless Architecture

## 📝 Topic: Serverless Compute for Automation, Cost Optimization & Security — A DevOps Perspective
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 15, 2026

---

## 🎯 Learning Objectives
* Understand what "serverless" actually means in practice, using Lambda as the example.
* Compare EC2 and Lambda directly on responsibility, scaling, and cost model.
* Learn the core DevOps use cases for Lambda: cost optimization, security/compliance auditing, routine tasks.
* Understand Lambda's practical building blocks — handler, triggers, environment variables, IAM Roles, Function URLs.
* Set up the foundational understanding needed before Day 18's hands-on Cloud Cost Optimization project.

---

## ⚡ Part 1 — What Is AWS Lambda?

### Definition

```
Lambda = a COMPUTE service, similar in category to EC2 —
         but operating on a SERVERLESS model
```

### What "Serverless" Actually Means

```
You are NOT responsible for:
  → Managing servers
  → OS patching
  → Provisioning hardware (CPU/RAM)

AWS handles ALL of this automatically, behind the scenes.

(Servers still physically exist somewhere — "serverless"
 means YOU never have to think about or manage them,
 not that servers have literally vanished)
```

### Event-Driven Execution

```
Lambda functions are TRIGGERED by events:
  → A file uploaded to S3
  → A CloudWatch schedule (e.g., "run every day at 2am")
  → An API call
  → Many other AWS service events
```

### Lifecycle

```
Event fires
    ↓
AWS automatically SCALES UP resources to run the function
    ↓
Function executes
    ↓
AWS automatically SCALES DOWN / tears down the environment
    once the task completes

→ No idle capacity sitting around between invocations
```

> **Why this lifecycle matters:** this is the direct architectural answer to the underutilization problem from AWS Day 01 (100GB provisioned, 1GB actually used). Lambda takes that principle to its logical extreme — resources exist ONLY for the exact duration of execution, then disappear entirely.

---

## ⚖️ Part 2 — Comparison: EC2 vs. Lambda

| Aspect | EC2 | Lambda |
|---|---|---|
| **Server management** | You control configuration, IPs, scaling policies, security patches | AWS handles all infrastructure |
| **What you provide** | An entire running instance | Just the code |
| **Cost responsibility** | You must stop/delete instances yourself to control cost | Strictly pay-as-you-USE |
| **Best fit** | Long-running, continuously active workloads | Intermittent, event-driven tasks |

```
Cost model contrast:

EC2:  Pay for the instance running, whether or not
      it's actively doing anything at that moment
      (recall: shutting down instances during non-working
       hours from the Day 03 EC2/Jenkins session)

Lambda: Pay ONLY for actual invocations and execution time
        → Zero cost when the function isn't running at all
```

> **The practical decision point:** if a workload runs continuously (a web server, a database), EC2 is the natural fit. If a workload runs briefly, occasionally, or in response to specific events (a daily cleanup script, a file-processing trigger), Lambda's pay-per-use model is dramatically more cost-efficient than paying for an always-on EC2 instance that sits idle most of the time.

---

## 🔧 Part 3 — DevOps Use Cases

### Cost Optimization

```
Automated identification and cleanup of unused resources:
  → Stale EBS volumes (not attached to any running instance)
  → Unattached Elastic IPs (still billed even when unused —
    directly connects to the Elastic IP limits/costs
    mentioned in the Day 32.1 architecture project)

Scheduled Lambda function:
  → Runs on a CloudWatch schedule (e.g., daily)
  → Scans the account for these wasted resources
  → Either deletes them automatically, or reports them
    for manual review
```

### Security & Compliance

```
Automated infrastructure AUDITING:
  → Flag non-compliant EBS volumes (e.g., older gp2 type
    instead of the more current gp3)
  → Identify S3 buckets with PUBLIC access enabled
    (a common, serious misconfiguration —
     connects directly to the static-site hosting bucket
     policy work from Day 09, where public access
     was intentionally enabled for a specific use case —
     the compliance check here is about catching
     UNINTENTIONAL public exposure)

Notification:
  → Findings sent via SNS (same service used for the
    CloudWatch alarm email alerts from Day 16)
```

### Routine Tasks

```
Maintenance scripts, daily reports, etc.
  → Run without maintaining a DEDICATED server
    just to execute a script for a few minutes a day
```

> **The unifying theme across all three use cases:** Lambda is DevOps' tool for "small, periodic, or reactive automation" — the moment a task doesn't need a persistently running server, Lambda is very likely the more efficient and lower-maintenance choice.

---

## 🛠️ Part 4 — Practical Implementation Highlights

### Supported Languages

```
Python, Node.js, Java, Go, Ruby
```

### The Lambda Handler

```python
def lambda_handler(event, context):
    # Main entry point — this is what gets invoked
    # when the function is triggered

    result = do_the_actual_work(event)
    return result

def do_the_actual_work(event):
    # Additional helper functions can be defined freely —
    # only the HANDLER itself needs the specific name
    # the trigger is configured to call
    pass
```

```
Default handler name: lambda_handler
  → The trigger calls THIS function specifically
  → Any number of additional helper functions can exist
    alongside it, called internally as needed
```

### Configuration Building Blocks

| Component | Purpose |
|---|---|
| **Environment Variables** | Pass configuration values into the code without hardcoding them |
| **IAM Role** | Grants the Lambda function permission to access other AWS services (S3, SNS, etc.) |
| **Triggers** | Configured via CloudWatch schedules, S3 events, and other AWS service events |
| **Function URL** | Optionally exposes the Lambda function via public HTTP access |

> **IAM Role here is the same Role pattern from IAM (Day 02) and CodeBuild/CodeDeploy (Days 14-15) yet again:** Lambda is a service, not a person, so it authenticates and is authorized via a Role — and that Role needs exactly the permissions the function's code actually needs (e.g., `s3:ListBucket`, `sns:Publish`), following the same least-privilege principle from Day 02.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS Lambda** | AWS's serverless compute service — runs code without managing servers |
| **Serverless** | A model where the cloud provider fully manages the underlying compute infrastructure |
| **Event-Driven** | Execution triggered by specific events rather than running continuously |
| **Pay-as-you-use** | A billing model charging only for actual execution time/invocations |
| **Lambda Handler** | The named entry-point function AWS invokes when the Lambda is triggered |
| **Environment Variable (Lambda)** | A configuration value passed into the function without hardcoding it in code |
| **Trigger** | The event source (S3, CloudWatch, API Gateway, etc.) that invokes a Lambda function |
| **Function URL** | An optional public HTTP endpoint for directly invoking a Lambda function |
| **Stale Resource** | An unused/unattached AWS resource (e.g., an unattached EBS volume or Elastic IP) still incurring cost |

---

## 📂 Summary of Tasks
- ✅ Understood: What "serverless" means — no server management, OS patching, or hardware provisioning.
- ✅ Understood: Lambda's event-driven, auto-scale-up/auto-tear-down execution lifecycle.
- ✅ Compared: EC2 (continuous, self-managed) vs. Lambda (intermittent, pay-per-use, fully managed).
- ✅ Learned: Cost optimization use cases — auto-detecting stale EBS volumes and unattached Elastic IPs.
- ✅ Learned: Security/compliance use cases — flagging non-compliant EBS volumes and publicly exposed S3 buckets.
- ✅ Learned: Routine task automation without maintaining a dedicated server.
- ✅ Reviewed: Lambda's supported languages and the `lambda_handler` entry-point pattern.
- ✅ Understood: Environment Variables, IAM Roles, Triggers, and Function URLs as Lambda's core configuration pieces.
- ✅ Noted: Today's session is the explicit prerequisite for Day 18's hands-on Cloud Cost Optimization project.

---

## 💡 My Takeaway

The Elastic IP and stale-EBS-volume cost optimization use case landed immediately, because it's directly connected to something flagged as a gotcha back on Day 32.1 — Elastic IPs are a limited, billed resource, and it's genuinely easy to leave one unattached and forgotten after tearing down a NAT Gateway or test instance. Seeing Lambda proposed as the automated fix for exactly that kind of drift (a scheduled function that finds and reports/cleans these up) makes the earlier gotcha feel like a solved problem rather than just a thing to remember to check manually.

The IAM Role requirement for Lambda is now the fourth or fifth time this exact "services authenticate via Roles, not Users, following least privilege" pattern has shown up — IAM itself (Day 02), CodeBuild (Day 14), CodeDeploy (Day 15), and now Lambda. At this point it's less a new concept each time and more confirmation that this really is THE standard AWS security pattern for any non-human AWS actor.

The EC2-vs-Lambda cost framing is a genuinely useful mental shortcut going forward: "does this workload need to run continuously, or does it run briefly/occasionally in response to something?" is now my first question when deciding where a given piece of automation belongs — continuous → EC2 (or a container), brief/event-driven → Lambda.

---


## 🔗 Resources
* [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
* [Lambda Execution Model](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
* [Lambda Function URLs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html)
* [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
* [IAM Roles for Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*