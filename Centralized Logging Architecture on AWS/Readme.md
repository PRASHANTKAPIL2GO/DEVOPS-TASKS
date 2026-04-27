# 📋 Centralized Logging Architecture on AWS

A production-ready AWS logging setup that collects logs from multiple sources — **VPC Flow Logs**, **ALB Access Logs**, and **CloudTrail** — into a single S3 bucket with lifecycle management for cost-effective long-term storage.

---

## 📌 What This Project Does

- Captures **all network traffic metadata** from a VPC using Flow Logs → sent to S3
- Collects **HTTP request logs** from an Application Load Balancer → sent to S3
- Stores **API activity audit logs** from CloudTrail → sent to S3
- Organizes all logs in a **single centralized S3 bucket** with separate folders per source
- Applies an **S3 Lifecycle rule** to automatically move older logs to cheaper storage classes

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Log Sources                                   │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌────────────────┐  │
│  │   VPC Flow Logs  │  │  ALB Access Logs │  │  CloudTrail    │  │
│  │  (vpc-03434c...) │  │  (prod-app-alb)  │  │  (API calls)   │  │
│  │  Traffic: All    │  │  HTTP:80         │  │  All regions   │  │
│  └────────┬────────┘  └────────┬────────┘  └───────┬────────┘  │
│           │                    │                    │           │
└───────────┼────────────────────┼────────────────────┼───────────┘
            │                    │                    │
            └────────────────────┼────────────────────┘
                                 │
                                 ▼
            ┌────────────────────────────────────────┐
            │     centralized-logs-bucket-2026        │
            │                                        │
            │  📁 vpc-flow-logs/                     │
            │  📁 alb-logs/                          │
            │  📁 cloudtrail-logs/                   │
            │                                        │
            │  Lifecycle Rule: centralized-log-       │
            │  retention-rule                        │
            │  → Transitions to Standard-IA          │
            └────────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │    S3 Lifecycle          │
                    │  Standard → Standard-IA  │
                    │  (cost optimization)     │
                    └─────────────────────────┘
```

---

## 🧩 AWS Resources

### S3 — Centralized Log Bucket

| Property | Value |
|---|---|
| **Bucket Name** | `centralized-logs-bucket-2026` |
| **Region** | Europe — Stockholm (`eu-north-1`) |
| **Total Folders** | 3 |

**Folder Structure:**

```
centralized-logs-bucket-2026/
├── alb-logs/          ← ALB HTTP access logs
├── cloudtrail-logs/   ← AWS API activity audit trail
└── vpc-flow-logs/     ← VPC network traffic metadata
```

Each log source writes into its own dedicated folder — this makes it easy to search, filter, and set different permissions per log type if needed later.

---

### S3 Lifecycle Rule — `centralized-log-retention-rule`

| Property | Value |
|---|---|
| **Status** | ✅ Enabled |
| **Scope** | Entire bucket |
| **Current version action** | Transition to `Standard-IA` |
| **Noncurrent version action** | Transition to `Standard-IA` |
| **Min object size for transitions** | 128 KB |

**What this means in plain English:** Logs that aren't accessed frequently get automatically moved from `S3 Standard` to `S3 Standard-IA` (Infrequent Access), which costs significantly less. You still keep everything — it just gets cheaper to store over time.

> 💡 Standard-IA costs ~58% less per GB than Standard, while still allowing instant retrieval when you need it.

---

### Log Source 1 — VPC Flow Logs

| Property | Value |
|---|---|
| **Flow Log Name** | `my-flowlog-1` |
| **Flow Log ID** | `fl-06d95f2741d01747a` |
| **VPC** | `vpc-03434cafac89b43b1` (Default VPC, `172.31.0.0/16`) |
| **Traffic Type** | All (captures ACCEPT, REJECT, and all traffic) |
| **Destination Type** | S3 |
| **Destination** | `centralized-logs-bucket-2026/vpc-flow-logs/` |

VPC Flow Logs capture metadata about every network connection in your VPC — source IP, destination IP, port, protocol, bytes transferred, and whether the traffic was accepted or rejected. Essential for security audits and network troubleshooting.

**Example flow log entry:**
```
2 660815084882 eni-abc123 10.0.1.5 172.217.0.0 54321 443 6 10 4000 1700000000 1700000060 ACCEPT OK
```

---

### Log Source 2 — Application Load Balancer

| Property | Value |
|---|---|
| **ALB Name** | `prod-app-alb` |
| **Status** | ✅ Active |
| **Type** | Application Load Balancer |
| **Scheme** | Internet-facing |
| **VPC** | `vpc-03434cafac89b43b1` |
| **Listener** | HTTP:80 → Forward to target group `random-1` (100%) |
| **Availability Zones** | `eu-north-1b` (eun1-az2), `eu-north-1a` (eun1-az1) |
| **DNS Name** | `prod-app-alb-1290291052.eu-north-1.elb.amazonaws.com` |
| **Hosted Zone** | `Z23TAZ6LKFMNIO` |
| **Created** | February 28, 2026 |
| **ARN** | `arn:aws:elasticloadbalancing:eu-north-1:660815084882:loadbalancer/app/prod-app-alb/373ea30d845cb3f4` |
| **Log Destination** | `centralized-logs-bucket-2026/alb-logs/` |

ALB access logs record every request the load balancer processes — client IP, request path, response code, latency, target, and more. Useful for debugging 4xx/5xx errors and analyzing traffic patterns.

---

### Log Source 3 — CloudTrail

CloudTrail records every AWS API call made in your account — who did what, when, from where. Logs land in the `cloudtrail-logs/` folder in the centralized bucket.

| What gets logged | Examples |
|---|---|
| Console actions | Creating/deleting EC2 instances, S3 uploads |
| CLI/SDK calls | `aws s3 cp`, `aws ec2 describe-instances` |
| IAM changes | Role creation, policy attachments |
| Network changes | Security group edits, VPC modifications |

---

## ⚙️ Setup Guide

### Step 1 — Create the S3 Bucket

```bash
aws s3api create-bucket \
  --bucket centralized-logs-bucket-2026 \
  --region eu-north-1 \
  --create-bucket-configuration LocationConstraint=eu-north-1
```

Create the folder structure:

```bash
aws s3api put-object --bucket centralized-logs-bucket-2026 --key vpc-flow-logs/
aws s3api put-object --bucket centralized-logs-bucket-2026 --key alb-logs/
aws s3api put-object --bucket centralized-logs-bucket-2026 --key cloudtrail-logs/
```

---

### Step 2 — Enable VPC Flow Logs

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-03434cafac89b43b1 \
  --traffic-type ALL \
  --log-destination-type s3 \
  --log-destination arn:aws:s3:::centralized-logs-bucket-2026/vpc-flow-logs/
```

Or in the Console:
1. **VPC → Your VPCs → Select VPC**
2. **Flow logs tab → Create flow log**
3. Traffic type: **All**
4. Destination: **S3 bucket**
5. S3 bucket ARN: `arn:aws:s3:::centralized-logs-bucket-2026/vpc-flow-logs/`

---

### Step 3 — Enable ALB Access Logs

1. Go to **EC2 → Load Balancers → prod-app-alb**
2. **Attributes tab → Edit**
3. Enable **Access logs**
4. S3 location: `centralized-logs-bucket-2026/alb-logs/`

The S3 bucket needs a bucket policy allowing the ALB service to write:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "elasticloadbalancing.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::centralized-logs-bucket-2026/alb-logs/*"
    }
  ]
}
```

---

### Step 4 — Enable CloudTrail

```bash
aws cloudtrail create-trail \
  --name central-audit-trail \
  --s3-bucket-name centralized-logs-bucket-2026 \
  --s3-key-prefix cloudtrail-logs \
  --include-global-service-events \
  --is-multi-region-trail

aws cloudtrail start-logging --name central-audit-trail
```

---

### Step 5 — Set Up the S3 Lifecycle Rule

Apply a lifecycle policy to automatically move logs to cheaper storage:

```json
{
  "Rules": [
    {
      "ID": "centralized-log-retention-rule",
      "Status": "Enabled",
      "Filter": { "Prefix": "" },
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "STANDARD_IA"
        }
      ],
      "NoncurrentVersionTransitions": [
        {
          "NoncurrentDays": 30,
          "StorageClass": "STANDARD_IA"
        }
      ]
    }
  ]
}
```

```bash
aws s3api put-bucket-lifecycle-configuration \
  --bucket centralized-logs-bucket-2026 \
  --lifecycle-configuration file://lifecycle.json
```

---

## 💰 Storage Cost Optimization

| Storage Class | Cost (per GB/month) | Use case |
|---|---|---|
| S3 Standard | ~$0.023 | Recent logs, actively queried |
| S3 Standard-IA | ~$0.0125 | Logs older than 30 days |
| S3 Glacier | ~$0.004 | Archive logs (90+ days) |
| S3 Glacier Deep Archive | ~$0.00099 | Long-term compliance (1+ year) |

> The lifecycle rule in this setup automatically handles the Standard → Standard-IA transition. You can extend it to add Glacier archiving for logs older than 90 days.

---

## 🔍 Querying Logs with Athena

Once logs are in S3, you can query them directly using **Amazon Athena** without moving or loading the data.

**Example — Find all rejected traffic in VPC Flow Logs:**
```sql
SELECT sourceaddress, destinationaddress, destinationport, action
FROM vpc_flow_logs
WHERE action = 'REJECT'
AND day = '2026/04/11'
LIMIT 100;
```

**Example — Find all 5xx errors from the ALB:**
```sql
SELECT request_url, target_status_code, response_processing_time
FROM alb_logs
WHERE elb_status_code LIKE '5%'
ORDER BY response_processing_time DESC
LIMIT 50;
```

---

## 🔐 Security Best Practices

- **Block public access** on `centralized-logs-bucket-2026` — logs should never be publicly readable
- Use **bucket policies** (not ACLs) to grant write access to AWS services (ALB, CloudTrail, VPC)
- Enable **S3 Object Lock** or versioning if you need tamper-proof audit logs for compliance
- Consider **KMS encryption** (`SSE-KMS`) for logs containing sensitive data
- Use **IAM policies** to restrict which users/roles can read from each log folder

---

## 🧹 Cleanup

```bash
# 1. Disable VPC Flow Logs
aws ec2 delete-flow-logs --flow-log-ids fl-06d95f2741d01747a

# 2. Disable ALB access logs (via Console → Attributes → Edit)

# 3. Stop CloudTrail
aws cloudtrail stop-logging --name central-audit-trail
aws cloudtrail delete-trail --name central-audit-trail

# 4. Empty and delete the S3 bucket
aws s3 rm s3://centralized-logs-bucket-2026 --recursive
aws s3 rb s3://centralized-logs-bucket-2026
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: Europe — Stockholm (`eu-north-1`)