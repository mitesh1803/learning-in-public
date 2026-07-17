![Progress](https://img.shields.io/badge/Progress-37%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 18 — Cloud Cost Optimization Project (Lambda + Boto3)

## 📝 Topic: Event-Driven Serverless Cleanup of Stale EBS Snapshots
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 16, 2026
---

## 🎯 Learning Objectives
* Understand why cost optimization is a core DevOps/Cloud Engineer responsibility.
* Understand how stale resources (EBS volumes, snapshots) accumulate and quietly rack up cost.
* Build a real Lambda function using Python/boto3 to detect orphaned EBS snapshots.
* Apply least-privilege IAM permissions specifically scoped to this function's actual needs.
* Schedule the cleanup via CloudWatch Events (EventBridge).
* Understand when automatic deletion is appropriate vs. when an SNS notification is the safer choice.

---

## 💸 Part 1 — Why Cost Optimization Matters

### The Business Context

```
Startups and mid-scale orgs move to the cloud specifically
to AVOID the heavy cost and management overhead of
maintaining on-premises data centers
  (direct callback to AWS Day 01's on-premises vs.
   public cloud comparison)

But moving to the cloud doesn't automatically mean
efficient spending — unmanaged cloud costs can
quietly balloon just as easily as on-prem overhead did.
```

### How Stale Resources Accumulate

```
Common developer workflow:
  1. Create an EC2 instance for testing
  2. Attach an EBS volume to it
  3. Take a snapshot of that volume (for backup/testing)
  4. Delete the EC2 instance when done testing
        ↓
  PROBLEM: the EBS volume and/or its snapshot are
  sometimes left behind, completely disconnected
  from any active instance
        ↓
  → Continues to incur AWS charges indefinitely,
    with nobody actively using or even remembering it exists
```

> **Why this specific pattern is so common:** deleting an instance feels like "cleaning up," but EBS volumes and snapshots are separate billable resources that don't automatically get deleted alongside the instance unless explicitly configured to do so (or manually removed afterward). This is exactly the "unattached Elastic IP" problem flagged on Day 17, just showing up again with a different resource type.

---

## 🏗️ Part 2 — Project Architecture: Event-Driven Serverless Cleanup

### The Goal

```
Automatically IDENTIFY and DELETE "stale" EBS snapshots
  → snapshots no longer associated with any active
    volume or EC2 instance
```

### Tools Used

| Tool | Role |
|---|---|
| **AWS Lambda** | Runs the actual cleanup logic (Python/boto3) |
| **AWS CloudWatch (Events/EventBridge)** | Triggers the Lambda function on a recurring schedule |
| **Python & boto3** | AWS SDK for Python — used to fetch, filter, and delete resources via API calls |

```
Full flow:
  CloudWatch schedule fires (e.g., daily at 2am)
        ↓
  Lambda function invoked
        ↓
  boto3 calls: describe_instances(), describe_volumes(),
               describe_snapshots()
        ↓
  Compare results → identify ORPHANED snapshots
        ↓
  delete_snapshot() called on each orphaned snapshot
```

---

## 🛠️ Part 3 — Key Implementation Steps

### Step 1: Define the Logic

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # Get all current snapshots owned by this account
    snapshots = ec2.describe_snapshots(OwnerIds=['self'])['Snapshots']

    # Get all currently existing volumes
    volumes = ec2.describe_volumes()['Volumes']
    active_volume_ids = {v['VolumeId'] for v in volumes}

    orphaned_snapshots = []
    for snapshot in snapshots:
        volume_id = snapshot.get('VolumeId')
        # A snapshot is "orphaned" if its source volume
        # no longer exists at all
        if volume_id not in active_volume_ids:
            orphaned_snapshots.append(snapshot['SnapshotId'])

    for snapshot_id in orphaned_snapshots:
        ec2.delete_snapshot(SnapshotId=snapshot_id)
        print(f"Deleted orphaned snapshot: {snapshot_id}")

    return {
        'deleted_count': len(orphaned_snapshots),
        'deleted_snapshots': orphaned_snapshots
    }
```

```
Core logic:
  1. Describe current INSTANCES, VOLUMES, and SNAPSHOTS
  2. Compare snapshot → source volume relationships
  3. Any snapshot whose source volume no longer exists
     = ORPHANED → flagged for deletion
```

### Step 2: IAM Permissions (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DescribeSnapshots",
        "ec2:DeleteSnapshot"
      ],
      "Resource": "*"
    }
  ]
}
```

> **Exactly four permissions, nothing more:** the function needs to READ instance/volume/snapshot state and DELETE snapshots — that's it. No `ec2:TerminateInstances`, no broader `ec2:*` wildcard. This is the least-privilege principle from IAM (Day 02) applied concretely: the Lambda's execution Role gets precisely what the code in Step 1 actually calls, nothing broader "just in case."

### Step 3: Deployment

```
Lambda Console configuration:
  → Runtime: Python 3.x
  → Timeout: set appropriately (e.g., 10 seconds —
    enough for the describe/delete API calls to complete,
    without leaving it needlessly long)
  → Attach the IAM Role from Step 2
```

### Step 4: Scheduling

```
CloudWatch Events (EventBridge) Rule:
  → Schedule expression: e.g., rate(1 day) or a cron expression
  → Target: this Lambda function

  → Runs the cleanup automatically, on a recurring basis,
    with zero manual intervention needed going forward
```

---

## ✅ Part 4 — Best Practices

### Least Privilege

```
Reinforced again: grant ONLY the specific permissions
needed for the task — no broader access "to be safe,"
since that inverts the actual security goal.
```

### Testing Before Automation

```
CRITICAL step: manually trigger the Lambda using a
TEST EVENT before enabling the CloudWatch schedule

Why: this is a DESTRUCTIVE operation (delete_snapshot).
     Confirming the logic correctly identifies ONLY
     truly orphaned snapshots — and not, say, snapshots
     that are legitimately being used for cross-region
     backup purposes — BEFORE it runs unattended and
     automatically is essential.
```

### Notification Alternative for Production

```
In more cautious production environments:
  → Instead of directly calling delete_snapshot(),
    send an SNS notification listing candidate
    stale resources for a human to manually review

  → Trades some automation convenience for a safety
    checkpoint before anything irreversible happens —
    a reasonable tradeoff for resources where a
    false positive could mean losing a needed backup
```

> **Why this alternative matters:** automatic deletion is appropriate for low-risk, well-understood cleanup (e.g., in a personal learning/test account). In a real production environment, the cost of a false-positive deletion (losing a snapshot that turns out to matter) can be much higher than the cost savings from full automation — hence the SNS-notify-and-review pattern as the safer default there.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Stale/Orphaned Resource** | An AWS resource (volume, snapshot, IP) no longer associated with any active resource, still incurring cost |
| **boto3** | The official AWS SDK for Python, used to make API calls from code |
| **`describe_snapshots` / `describe_volumes`** | boto3 EC2 client methods that list existing resources for comparison |
| **`delete_snapshot`** | The boto3 method that deletes a specific EBS snapshot by ID |
| **CloudWatch Events (EventBridge)** | The scheduling mechanism used to trigger the Lambda function on a recurring basis |
| **Test Event** | A manually-triggered Lambda invocation used to validate logic before enabling automated scheduling |
| **Least Privilege IAM Policy** | A policy scoped to exactly the actions the function's code performs, nothing broader |

---

## 📂 Summary of Tasks
- ✅ Understood: Why cost optimization is a core, ongoing DevOps responsibility, not a one-time setup task.
- ✅ Understood: How stale EBS volumes and snapshots accumulate after instance deletion.
- ✅ Wrote: A Python/boto3 Lambda function to identify snapshots orphaned from any active volume.
- ✅ Scoped: A least-privilege IAM policy with exactly four required EC2 permissions.
- ✅ Configured: The Lambda function's runtime, timeout, and IAM Role.
- ✅ Scheduled: The function via CloudWatch Events (EventBridge) for recurring automated runs.
- ✅ Practiced: Manually testing the function with a test event before enabling the automated schedule.
- ✅ Considered: The SNS-notification alternative to direct deletion for higher-caution production use.

---

## 💡 My Takeaway

This project felt like the natural payoff of the Day 17 Lambda concepts — going from "here's what serverless means and why it's cost-efficient" to actually writing the boto3 logic that finds and removes waste. The `describe_*` → compare → `delete_*` pattern is genuinely simple once broken down, and it's a real, directly reusable template — I can already picture pointing a near-identical script at unattached Elastic IPs or unused Load Balancers in my own AWS learning account, rather than manually auditing the Console periodically.

The "test manually before scheduling" best practice is the one I want to treat as completely non-negotiable for anything involving `delete_*` calls specifically — this is a case where the standard "move fast, iterate" instinct needs to be overridden by "confirm this destroys exactly what I expect, and nothing else, before it ever runs unattended." The SNS-notification-instead-of-auto-delete alternative is a good middle ground worth defaulting to for anything I'm not 100% confident about long-term, especially outside a disposable practice account.

Seeing the least-privilege IAM policy scoped to exactly four permissions was a clean, concrete example of something I now understand at a much more visceral level than back on Day 02 — writing the actual policy JSON myself, matching it precisely against what the code needs, made "least privilege" feel like a specific engineering discipline rather than just a security buzzword.

---

## 🔗 Resources
* [AWS Lambda Documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
* [Boto3 EC2 Client Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/ec2.html)
* [Amazon EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)
* [Amazon EventBridge Documentation](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-what-is.html)
* [AWS IAM Least Privilege Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*