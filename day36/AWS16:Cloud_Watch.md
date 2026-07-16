![Progress](https://img.shields.io/badge/Progress-36%25-orange?style=for-the-badge&logo=progress)

# 🚀 AWS Day 16 — AWS CloudWatch Deep Dive

## 📝 Topic: Monitoring, Metrics, Alarms, and Log Insights
**Instructor:** [Abhishek Veeramalla](https://github.com/iam-veeramalla)

**Date:** July 14, 2026
---

## 🎯 Learning Objectives
* Understand CloudWatch's role as AWS's monitoring, alerting, and logging "gatekeeper."
* Learn CloudWatch's six core features: Monitoring, Metrics, Alarms, Logging, Custom Metrics, and Cost/Scaling integration.
* Explore Log Groups and Log Insights using CodeBuild logs as a real example.
* Simulate CPU load on EC2 and observe real-time metric tracking.
* Configure a CloudWatch Alarm tied to an SNS email notification.
* Understand the distinction between default and custom metrics.

---

## 👁️ Part 1 — CloudWatch's Role

```
CloudWatch = the "gatekeeper" for AWS —
             manages monitoring, alerting, reporting, and logging

Core value: provides insight into your infrastructure
            even when you are NOT actively watching it
```

> **Why this framing matters:** the whole point of a monitoring service is to remove the need for a human to be staring at a dashboard 24/7. CloudWatch's job is to notice things on your behalf and only interrupt you when something actually needs attention.

---

## 🧩 Part 2 — Core Features

### 1. Monitoring

```
Tracks infrastructure HEALTH across AWS resources
  e.g., is this EC2 instance up? Responding? Healthy?
```

### 2. Metrics

```
QUANTIFIABLE data points, tracked over time:
  → CPU utilization
  → API request counts
  → Memory usage

Metrics are the raw signal used to communicate
"performance status" — numbers, not narratives.
```

### 3. Alarms

```
ACTIONS triggered by metrics crossing a defined threshold

  Metric > Threshold → Alarm state changes → Action fires
    (notification, automated response, etc.)
```

### 4. Logging (Log Insights)

```
Automatically RECORDS system activities:
  e.g., which service accessed an S3 bucket or EC2 instance

Log Insights = a query interface for searching/analyzing
               those recorded logs after the fact
```

### 5. Custom Metrics

```
Extends monitoring BEYOND the ~1,000+ metrics AWS
tracks by default — e.g.:
  → Memory usage (NOT tracked by default on EC2 —
    a common surprise for engineers new to CloudWatch)
  → Custom application-level performance indicators
```

### 6. Cost Optimization & Scaling

```
CloudWatch integrates with:
  → AWS Lambda (trigger functions based on metrics/events)
  → Auto Scaling Groups (scale instance count based on
    real-time metric thresholds — directly connects to
    the ASG concept from the Day 32.1 production architecture)
```

---

## 📜 Part 3 — Practical Demonstration: Log Groups & Insights

### CodeBuild Logs as a Working Example

```
CloudWatch automatically creates a LOG GROUP for
AWS services like CodeBuild — no manual setup required.

Key property: these logs PERSIST even if the underlying
resource is later DELETED
  (e.g., delete a CodeBuild project → its historical
   build logs remain accessible in CloudWatch)
```

> **Why this persistence matters:** it enables historical auditing — reviewing why a build failed weeks ago, or confirming when a specific successful build actually ran — completely independent of whether the resource that generated those logs still exists. This connects directly to the CodeBuild debugging work from Day 14 — those exact build logs are what CloudWatch was capturing the whole time.

---

## 🔥 Part 4 — Practical Demonstration: Real-Time EC2 Metrics & Alarming

### Simulating Load

```python
# CPU_spike.py — simulates CPU load on the EC2 instance
import time

def burn_cpu():
    while True:
        pass  # busy-loop to consume CPU cycles

burn_cpu()
```

```bash
python3 CPU_spike.py
```

### Watching the Metric

```
CloudWatch Console → Metrics → EC2 → CPUUtilization
  → Observe the value climbing in near real-time
    as the script consumes CPU cycles
```

### Configuring the Alarm

```
CloudWatch Console → Alarms → Create Alarm
  Metric: CPUUtilization
  Condition: Threshold >= 50%
  Action: Notify via SNS Topic
```

### SNS Integration for Email Alerts

```
Amazon SNS (Simple Notification Service):
  → Create a Topic
  → Add an email SUBSCRIPTION to that topic
  → CRITICAL STEP: the subscribed email address must
    CONFIRM the subscription (via a confirmation email)
    before SNS will actually deliver notifications to it

Alarm → ALARM state → publishes to SNS Topic → email sent
```

> **The confirmation step is easy to forget and silently breaks the whole demo:** if the email subscription was never confirmed, the alarm can fire perfectly correctly and the SNS topic can publish successfully — but no email actually arrives, with no obvious error surfaced anywhere. Worth checking subscription status FIRST if an alarm "isn't sending notifications."

---

## ✅ Part 5 — Key Takeaways for DevOps

### Default vs. Custom Metrics

```
CloudWatch tracks 1,000+ metrics BY DEFAULT
  → but DevOps engineers must often implement
    CUSTOM metrics for specific needs

Classic example: EC2 memory usage is NOT a default metric
  → Requires installing the CloudWatch Agent on the
    instance and configuring it to publish memory
    as a custom metric
```

### Proactive Management

```
Use Alarms to REDUCE the need for 24/7 manual oversight:
  → Let the SYSTEM notify you when a threshold is breached
  → This is the practical implementation of the
    "gatekeeper" framing from Part 1 — CloudWatch
    watches so a human doesn't have to
```

### Dashboarding

```
CloudWatch Dashboards provide a CENTRALIZED view
of multiple metrics at once
  → Efficient visual monitoring of overall system health
  → One screen instead of navigating between many
    individual metric pages
```

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **CloudWatch** | AWS's monitoring, alerting, and logging service |
| **Metric** | A quantifiable, time-tracked data point (CPU, memory, request count, etc.) |
| **Alarm** | An action triggered when a metric crosses a defined threshold |
| **Log Group** | A container for logs automatically created by AWS services, persisting independently of the source resource |
| **Log Insights** | CloudWatch's query interface for searching and analyzing collected logs |
| **Custom Metric** | A metric not tracked by default, published via the CloudWatch Agent or application code |
| **CPUUtilization** | A default EC2 metric measuring CPU load as a percentage |
| **Amazon SNS** | Simple Notification Service — used here to deliver email alerts when an alarm fires |
| **SNS Subscription Confirmation** | The required opt-in step for an email address before SNS will deliver notifications to it |
| **CloudWatch Dashboard** | A centralized visual view combining multiple metrics into one screen |

---

## 📂 Summary of Tasks
- ✅ Understood: CloudWatch's role as AWS's monitoring/alerting/logging "gatekeeper."
- ✅ Learned: The six core CloudWatch features — Monitoring, Metrics, Alarms, Logging, Custom Metrics, Cost/Scaling integration.
- ✅ Explored: Automatically created Log Groups for CodeBuild, and confirmed logs persist after resource deletion.
- ✅ Simulated: CPU load on an EC2 instance via a Python script and observed the CPUUtilization metric climb in real time.
- ✅ Configured: A CloudWatch Alarm triggering at 50% CPU utilization.
- ✅ Integrated: Amazon SNS for email notifications, and confirmed the subscription-confirmation requirement.
- ✅ Understood: The default-vs-custom metrics distinction, using EC2 memory usage as the classic example.
- ✅ Reinforced: Alarms as the mechanism enabling proactive management without 24/7 manual watching.
- ✅ Learned: Dashboards as a centralized, efficient view of overall system health.

---

## 💡 My Takeaway

The SNS subscription confirmation step is the detail I most want to remember — it's a perfect example of a failure that produces NO error anywhere in the pipeline. The alarm fires correctly, SNS publishes correctly, and yet nothing arrives, because a single email confirmation link was never clicked. This is now the fourth or fifth time in this series I've hit a version of "everything appears configured correctly, but one small unconfirmed/unchecked step silently breaks the whole thing" (Security Groups, Privileged Mode, container cleanup, and now SNS confirmation) — clearly a very common category of real-world infrastructure bug, not a one-off quirk.

The Log Groups persisting after resource deletion detail connects nicely back to the CodeBuild debugging work from Day 14 — I now understand that those build logs I was reading during troubleshooting were actually being captured by CloudWatch the entire time, independent of the CodeBuild project's own lifecycle. That's a good mental model to carry forward: CloudWatch logging isn't something you have to separately "remember to enable" for most AWS services — it's often already there, automatically, waiting to be queried when needed.

The custom-metrics gap (EC2 memory usage isn't tracked by default) is a genuinely useful thing to know upfront rather than discover mid-incident — if memory pressure is ever suspected as a root cause on a production EC2 instance, the CloudWatch Agent needs to already be installed and configured BEFORE that data becomes available, not after the fact.

---

## 📈 Next Up
**AWS Day 17:** CloudWatch Dashboards and Custom Metrics deep dive — installing the CloudWatch Agent to track memory usage, and building a centralized dashboard for the full CI/CD pipeline's infrastructure.

---

## 🔗 Resources
* [Amazon CloudWatch Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
* [CloudWatch Alarms Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
* [CloudWatch Agent (Custom Metrics)](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
* [CloudWatch Logs Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AnalyzingLogData.html)
* [Amazon SNS Documentation](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*