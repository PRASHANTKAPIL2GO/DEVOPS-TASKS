# 🚨 AWS CloudWatch CPU Alerting — SNS Email Notifications

A monitoring setup on AWS that automatically sends an **email alert** when an EC2 instance's CPU usage crosses a threshold — using **CloudWatch Alarms** connected to an **SNS Topic** with an email subscription.

---

## 📌 What This Project Does

- Monitors a **web-app EC2 instance** for CPU usage spikes
- Triggers a **CloudWatch Alarm** when CPU crosses the defined threshold
- Publishes an alert to an **SNS Topic** (`CPU-Alert-Topic`)
- Sends an **email notification** to a subscribed address automatically
- All status checks on the monitored instance are passing ✅

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────┐
│         EC2: web-app                │
│   i-093080dc9a3046573               │
│   Public IP: 51.20.7.246            │
│   t3.micro | eu-north-1             │
│   Status: ✅ Running                │
└──────────────┬──────────────────────┘
               │
               │ CPU Metrics (every 1-5 min)
               ▼
┌─────────────────────────────────────┐
│        CloudWatch Alarm             │
│   Trigger: CPU > threshold%         │
│   Region: us-east-1                 │
└──────────────┬──────────────────────┘
               │
               │ Alarm → ALARM state
               ▼
┌─────────────────────────────────────┐
│         SNS Topic                   │
│   CPU-Alert-Topic                   │
│   ARN: arn:aws:sns:us-east-1:       │
│   660815084882:CPU-Alert-Topic      │
│   Type: Standard                    │
└──────────────┬──────────────────────┘
               │
               │ EMAIL protocol
               ▼
┌─────────────────────────────────────┐
│         Email Subscriber            │
│   S24CSEU2033@bennet.edu.in         │
│   Status: Pending confirmation ⏳   │
└─────────────────────────────────────┘
```

> ⚠️ **Important**: The subscription status is **Pending confirmation** — the subscriber must click the confirmation link in the AWS email before alerts will be delivered.

---

## 🧩 AWS Resources

### EC2 Instance — web-app

| Property | Value |
|---|---|
| **Instance Name** | `web-app` |
| **Instance ID** | `i-093080dc9a3046573` |
| **Instance Type** | `t3.micro` |
| **State** | ✅ Running |
| **Public IP** | `51.20.7.246` |
| **Private IP** | `172.31.44.220` |
| **Public DNS** | `ec2-51-20-7-246.eu-north-1.compute.amazonaws.com` |
| **VPC** | `vpc-03434cafac89b43b1` (`new-vpc`) |
| **Subnet** | `subnet-01b7afc53ad4ad84f` |
| **IAM Role** | None |
| **IMDSv2** | Required |
| **Region** | Europe — Stockholm (`eu-north-1`) |

**Status Checks (all passing ✅):**

| Check | Status |
|---|---|
| System status check | ✅ Check passed |
| Instance status check | ✅ Check passed |
| EBS status check | ✅ Check passed |

---

### SNS Topic — CPU-Alert-Topic

| Property | Value |
|---|---|
| **Topic Name** | `CPU-Alert-Topic` |
| **Display Name** | `CPU-Alert-Topic` |
| **Type** | Standard |
| **ARN** | `arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic` |
| **Owner** | `660815084882` |
| **Region** | US East — N. Virginia (`us-east-1`) |

---

### SNS Subscription

| Property | Value |
|---|---|
| **Subscription ID** | `222791eb-4eb3-4306-95a4-5d665e5c6a33` |
| **Protocol** | EMAIL |
| **Endpoint** | `S24CSEU2033@bennet.edu.in` |
| **Status** | ⏳ Pending confirmation |
| **Topic** | `CPU-Alert-Topic` |
| **Principal** | `arn:aws:iam::660815084882:root` |

---

## ⚙️ Setup Guide

### Step 1 — Launch or Identify the EC2 Instance to Monitor

The `web-app` instance (`i-093080dc9a3046573`) in `eu-north-1` is the target being monitored. Ensure it's running and CloudWatch monitoring is enabled (basic monitoring is on by default; detailed monitoring sends metrics every 1 minute instead of 5).

To enable detailed monitoring:
```bash
aws ec2 monitor-instances --instance-ids i-093080dc9a3046573 --region eu-north-1
```

---

### Step 2 — Create an SNS Topic

```bash
aws sns create-topic \
  --name CPU-Alert-Topic \
  --region us-east-1
```

Output:
```json
{
  "TopicArn": "arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic"
}
```

---

### Step 3 — Subscribe an Email Address

```bash
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic \
  --protocol email \
  --notification-endpoint S24CSEU2033@bennet.edu.in \
  --region us-east-1
```

> After running this, AWS sends a confirmation email to `S24CSEU2033@bennet.edu.in`. The subscription stays in **Pending confirmation** until the recipient clicks **"Confirm subscription"** in that email.

---

### Step 4 — Create a CloudWatch Alarm

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "High-CPU-web-app" \
  --alarm-description "Alert when CPU exceeds 70%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 70 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-093080dc9a3046573 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic \
  --ok-actions arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic \
  --treat-missing-data notBreaching \
  --region eu-north-1
```

**What each key parameter means:**

| Parameter | Value | Meaning |
|---|---|---|
| `--period` | `300` | Check every 5 minutes |
| `--threshold` | `70` | Trigger if CPU > 70% |
| `--evaluation-periods` | `2` | Must exceed threshold for 2 consecutive periods |
| `--alarm-actions` | SNS ARN | Send alert email when alarm fires |
| `--ok-actions` | SNS ARN | Send recovery email when CPU drops back down |

---

### Step 5 — Confirm the Email Subscription

The subscription won't work until confirmed. Check your inbox at `S24CSEU2033@bennet.edu.in` for an email from `no-reply@sns.amazonaws.com` with subject:

```
AWS Notification - Subscription Confirmation
```

Click **"Confirm subscription"** in that email. The status will change from **Pending confirmation** → **Confirmed**.

---

## 🧪 Testing the Alarm

To test that everything is wired up correctly without waiting for real CPU spikes, you can manually set the alarm to ALARM state:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "High-CPU-web-app" \
  --state-value ALARM \
  --state-reason "Testing the alarm" \
  --region eu-north-1
```

This will immediately trigger the SNS notification and send an email. Reset it after testing:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "High-CPU-web-app" \
  --state-value OK \
  --state-reason "Test complete" \
  --region eu-north-1
```

---

## 📧 What the Alert Email Looks Like

When the alarm fires, the subscriber receives an email like:

```
From: no-reply@sns.amazonaws.com
Subject: ALARM: "High-CPU-web-app" in EU (Stockholm)

You are receiving this email because your Amazon CloudWatch Alarm
"High-CPU-web-app" in the EU (Stockholm) region has entered the
ALARM state, because "CPUUtilization GreaterThanThreshold 70.0"
as of April 11, 2026 15:00:00 UTC.

Alarm Details:
- Name: High-CPU-web-app
- State Change: OK -> ALARM
- Reason: Threshold Crossed: 2 datapoints were greater than 70.0
- Instance ID: i-093080dc9a3046573
```

---

## 🔔 Alarm States Explained

| State | Meaning |
|---|---|
| `OK` | Metric is within the defined threshold — all good |
| `ALARM` | Metric has breached the threshold — notification sent |
| `INSUFFICIENT_DATA` | Not enough data to determine state (common on startup) |

---

## 💡 Common Add-ons

Once the basic setup is working, consider these enhancements:

**Add multiple notification channels:**
```bash
# Also notify via SMS
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic \
  --protocol sms \
  --notification-endpoint +91XXXXXXXXXX
```

**Add more alarms for the same instance:**
- Memory utilization (requires CloudWatch Agent)
- Disk usage (requires CloudWatch Agent)
- Network in/out spikes
- Status check failures

**Auto-remediation with Lambda:**
Connect the SNS topic to a Lambda function that automatically restarts the instance or scales it up when CPU is sustained high.

---

## 🧹 Cleanup

```bash
# 1. Delete the CloudWatch alarm
aws cloudwatch delete-alarms \
  --alarm-names "High-CPU-web-app" \
  --region eu-north-1

# 2. Unsubscribe email
aws sns unsubscribe \
  --subscription-arn arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic:222791eb-4eb3-4306-95a4-5d665e5c6a33

# 3. Delete the SNS topic
aws sns delete-topic \
  --topic-arn arn:aws:sns:us-east-1:660815084882:CPU-Alert-Topic

# 4. Terminate EC2 instance if no longer needed
aws ec2 terminate-instances \
  --instance-ids i-093080dc9a3046573 \
  --region eu-north-1
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: US East — N. Virginia (`us-east-1`) for SNS | Europe — Stockholm (`eu-north-1`) for EC2