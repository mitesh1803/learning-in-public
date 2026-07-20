![Progress](https://img.shields.io/badge/Progress-39%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 39 — AWS Config for Compliance

## 📝 Topic: Monitoring Resource Inventory & Enforcing Organizational Rules via Lambda-Backed Config Rules
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 19, 2026

---

## 🎯 Learning Objectives
* Understand what AWS Config does and why compliance monitoring matters operationally.
* Understand the "thumb rule" concept and what makes a resource non-compliant.
* Walk through AWS Config's event-driven evaluation workflow using Lambda.
* Set up a custom Config Rule end to end.
* Understand the required IAM permissions for a Config-evaluating Lambda function.
* Learn the `put_evaluations` pattern for reporting compliance status back to Config.
* Recognize how this same pattern extends to other resource types (e.g., S3 public access).

---

## 📋 Part 1 — Understanding AWS Config

### Purpose

```
AWS Config helps:
  → Track resource INVENTORY and CHANGES
  → Ensure infrastructure stays COMPLIANT with
    internal security and operational standards
```

### The Concept of "Thumb Rules"

```
Organizations define specific compliance rules, e.g.:
  "All EC2 instances MUST have detailed monitoring enabled"

Resources that DON'T meet the defined criteria
  → marked as NON-COMPLIANT
```

> **Why "thumb rules" is a useful framing:** these are usually simple, binary, easily-checkable conditions (enabled/disabled, present/absent) rather than complex judgment calls — which is exactly what makes them well-suited to automated evaluation rather than requiring a human to manually audit every resource.

### The Workflow

```
AWS Config OBSERVES changes to resources
        ↓
Can TRIGGER an event (e.g., invoke a Lambda function)
        ↓
Lambda EVALUATES whether the resource is compliant
        ↓
Result reported back to AWS Config
```

---

## 🛠️ Part 2 — Practical Demonstration

### Setup

```
Monitoring target: EC2 instance compliance
Specific check: is "detailed monitoring" ENABLED?
```

### Triggering Evaluations

```
When an EC2 instance is CREATED or MODIFIED:
  → AWS Config triggers a Lambda function
  → That function EVALUATES the current state
```

### Troubleshooting

```
If an evaluation doesn't reflect correctly:
  1. Check CloudWatch Logs for the Lambda function
     (same CloudWatch logging pattern from Day 16
      and the ECS logging setup from Day 21)
  2. Increase the Lambda TIMEOUT DURATION if tasks
     are timing out (e.g., 3 seconds → 10 seconds)
```

> **Why the timeout adjustment matters specifically:** Lambda's default timeout is short, and API calls to `describe` resources plus the `put_evaluations` call back to Config can occasionally take longer than the default allows — especially under cold-start conditions. Bumping the timeout is a small, easy fix for what would otherwise look like a mysteriously "silent" evaluation failure.

---

## ⚙️ Part 3 — Setting Up a Custom Config Rule

### Step-by-Step

```
1. Navigate to AWS Config → Rules

2. Choose ONE of:
     → "Create Lambda rule" (for CUSTOM logic)
     → An AWS MANAGED rule (for predefined,
       common standards — no custom code needed)

3. Provide the Lambda function's ARN

4. Select the trigger type:
     → "When configuration changes"
       (real-time — evaluates immediately on any change)
     → "Periodic"
       (scheduled — evaluates on a recurring interval,
        similar in spirit to the CloudWatch/EventBridge
        scheduling from Day 18's cost optimization project)

5. Define the RESOURCE TYPE to watch
   (e.g., EC2 Instance)
```

> **Managed rule vs. custom Lambda rule — the decision point:** if the compliance check is a common, well-known standard (e.g., "S3 buckets must not be public"), an AWS managed rule likely already exists and requires zero custom code. Custom Lambda rules are for organization-SPECIFIC thumb rules that don't map to an existing managed rule — like this session's specific "EC2 detailed monitoring" check.

---

## 🐍 Part 4 — The Lambda Function & Permissions

### Implementation Pattern

```python
import boto3

def lambda_handler(event, context):
    config_client = boto3.client('config')
    ec2_client = boto3.client('ec2')

    invoking_event = event['invokingEvent']
    configuration_item = invoking_event['configurationItem']
    instance_id = configuration_item['resourceId']

    # Fetch current state of the resource
    response = ec2_client.describe_instances(InstanceIds=[instance_id])
    instance = response['Reservations'][0]['Instances'][0]

    # Compare against the desired compliance state
    monitoring_state = instance['Monitoring']['State']
    is_compliant = monitoring_state == 'enabled'

    compliance_type = 'COMPLIANT' if is_compliant else 'NON_COMPLIANT'

    # Push the result back to AWS Config
    config_client.put_evaluations(
        Evaluations=[
            {
                'ComplianceResourceType': 'AWS::EC2::Instance',
                'ComplianceResourceId': instance_id,
                'ComplianceType': compliance_type,
                'OrderingTimestamp': configuration_item['configurationItemCaptureTime']
            }
        ],
        ResultToken=event['resultToken']
    )
```

```
Core logic:
  1. Receive the EVENT PAYLOAD from AWS Config
  2. Use boto3 to FETCH the resource's actual current state
  3. COMPARE against the desired compliance condition
     (here: is monitoring enabled?)
  4. Report the result via put_evaluations()
```

### Required Permissions

```
Lambda execution role needs:
  → CloudWatchFullAccess     (logging + monitoring access)
  → EC2FullAccess            (describe/inspect EC2 resources)
  → ConfigRoleAccess         (report evaluation results
                               back to AWS Config)
```

> **Worth flagging as a gap in the strict least-privilege discipline from earlier sessions:** using "FullAccess" managed policies here (rather than scoping down to specific `ec2:Describe*` actions, as was done precisely on Day 18's cost-optimization Lambda) is a looser permission model than the series' own stated best practice. In a real production Config rule, tightening these to the specific `Describe`/`Get`/`config:PutEvaluations` actions actually used would be the more disciplined approach — worth doing myself even if the walkthrough used the broader managed policies for simplicity.

### The `put_evaluations` Method

```
This is THE mechanism that reports compliance status
back to AWS Config:
  → ComplianceResourceType  → what kind of resource
  → ComplianceResourceId    → which specific resource
  → ComplianceType          → COMPLIANT / NON_COMPLIANT
  → OrderingTimestamp       → when this state was captured
  → ResultToken              → correlates this response
                                back to the specific
                                Config evaluation request
```

---

## 🔄 Part 5 — Extending the Concept

```
This EXACT pattern generalizes to other resource types:

Example: S3 bucket public access
  → Write a similar Lambda function
  → Check whether "Block Public Access" is enabled
    on each bucket
  → Update the Config Rule's resource filter to
    target S3 buckets instead of EC2 instances

  → Same describe → compare → put_evaluations() shape,
    just pointed at a different resource and a
    different compliance condition
```

> **Direct connection to Day 09 and Day 19:** this is genuinely the automated version of manually checking whether an S3 bucket's public access setting is intentional (as it was for the Day 09 static site) versus accidental (a real security risk flagged conceptually back on Day 17's security/compliance Lambda use cases). AWS Config + this Lambda pattern is how that manual vigilance becomes continuous and automatic instead of a one-time check.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **AWS Config** | AWS's service for tracking resource inventory, changes, and compliance status |
| **Thumb Rule** | A simple, binary organizational compliance condition (e.g., "monitoring must be enabled") |
| **Config Rule** | A defined rule (managed or custom Lambda-backed) that evaluates resource compliance |
| **Managed Rule** | An AWS-provided, predefined Config Rule requiring no custom code |
| **Custom Lambda Rule** | A Config Rule backed by a custom Lambda function for organization-specific logic |
| **`put_evaluations`** | The boto3 Config client method used to report a resource's compliance status |
| **`ResultToken`** | A token correlating a Lambda's evaluation response to the specific Config request that triggered it |
| **Trigger Type: "When configuration changes"** | Real-time evaluation triggered immediately on resource change |
| **Trigger Type: "Periodic"** | Scheduled, recurring evaluation independent of specific changes |

---

## 📂 Summary of Tasks
- ✅ Understood: AWS Config's role in tracking resource inventory and enforcing compliance.
- ✅ Learned: The "thumb rule" concept and what marks a resource non-compliant.
- ✅ Walked through: The observe → trigger → evaluate → report Config workflow.
- ✅ Practiced: Monitoring EC2 detailed-monitoring compliance as a concrete example.
- ✅ Debugged: Evaluation failures via CloudWatch Logs and Lambda timeout adjustment.
- ✅ Set up: A custom Config Rule — Lambda ARN, trigger type, and resource type.
- ✅ Wrote: A Lambda function using boto3 to fetch, compare, and report compliance state.
- ✅ Noted: The required (though somewhat broad) IAM permissions for the Config-evaluating Lambda.
- ✅ Recognized: How this pattern extends directly to other resources, like S3 public access.

---

## 💡 My Takeaway

This session is the clearest example yet of the "Lambda for governance and automation" use case first introduced conceptually on Day 17 — AWS Config is essentially a purpose-built framework for exactly that pattern: observe a resource, run a small function to check it against a rule, report the result. Having already built the cost-optimization Lambda on Day 18 made writing this compliance-checking Lambda feel genuinely familiar rather than new — same `describe_*` → compare → act shape, just reporting a compliance verdict via `put_evaluations` instead of calling `delete_snapshot`.

I want to flag something for myself rather than just accept it: the `CloudWatchFullAccess`/`EC2FullAccess` permissions used in this walkthrough are noticeably looser than the four-permission, tightly-scoped IAM policy built for the Day 18 cost-optimization Lambda. That's worth treating as a "good enough for a tutorial, not good enough for production" gap — if I build this pattern for real, scoping down to the specific `ec2:Describe*` and `config:PutEvaluations` actions actually used would bring it in line with the least-privilege discipline I've been practicing consistently everywhere else in this series.

The S3-public-access extension mentioned at the end is the one I most want to actually build — it directly closes the loop between the "unintentional public S3 bucket" security risk flagged back on Day 17 and a genuine automated, continuous safeguard against it, rather than relying on remembering to check manually.

---


## 🔗 Resources
* [AWS Config Documentation](https://docs.aws.amazon.com/config/latest/developerguide/WhatIsConfig.html)
* [AWS Config Managed Rules List](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
* [Developing a Custom Config Rule with Lambda](https://docs.aws.amazon.com/config/latest/developerguide/gettingstarted-concepts.html)
* [`put_evaluations` API Reference (boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/config.html#ConfigService.Client.put_evaluations)
* [AWS IAM Least Privilege Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*