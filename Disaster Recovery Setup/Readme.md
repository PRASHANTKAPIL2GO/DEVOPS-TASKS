# 🛡️ AWS Disaster Recovery — S3 Cross-Region Replication & RDS Read Replica

A disaster recovery (DR) setup on AWS that protects both your **object storage** and your **relational database** using two core AWS features: S3 Cross-Region Replication (CRR) for files and RDS Read Replicas for database resilience.

---

## 📌 What This Project Does

- Automatically **replicates S3 objects** from a source bucket to a destination bucket in another region — so files survive a regional outage
- Maintains a **live database replica** (`read-replica-db`) that stays in sync with the primary RDS instance (`database-2`) in real time
- Demonstrates a practical, low-cost DR strategy without needing complex failover automation

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Disaster Recovery Architecture                    │
│                                                                     │
│  ┌──────────────────────────┐        ┌──────────────────────────┐   │
│  │   S3: mapp-dr-bucket     │──CRR──▶│   S3: destiation3        │   │
│  │   (Source)               │        │   (Destination)           │   │
│  │   Region: eu-north-1     │        │   Region: eu-north-1      │   │
│  │   Rule: crr-rule ✅      │        │   1 object replicated     │   │
│  │   Scope: Entire bucket   │        │   Screenshot(28).png      │   │
│  │   Storage: Same as src   │        │   356.3 KB — Standard     │   │
│  └──────────────────────────┘        └──────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────┐        ┌──────────────────────────┐   │
│  │   RDS: database-2        │──Rep──▶│   RDS: read-replica-db   │   │
│  │   Role: Primary          │        │   Role: Replica           │   │
│  │   Engine: MySQL          │        │   Engine: MySQL           │   │
│  │   Size: db.t3.micro      │        │   Size: db.t3.micro       │   │
│  │   AZ: eu-north-1b        │        │   AZ: eu-north-1b         │   │
│  │   CPU: 3.56%             │        │   Replication: ✅ Active  │   │
│  │                          │        │   Lag: 0 Seconds          │   │
│  └──────────────────────────┘        └──────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🪣 Part 1 — S3 Cross-Region Replication (CRR)

### What is CRR?

Cross-Region Replication automatically copies every new object written to a source bucket into a destination bucket — in real time, asynchronously. If the source region goes down, your data is already safe in the destination.

### Source Bucket — `mapp-dr-bucket`

| Property | Value |
|---|---|
| **Bucket Name** | `mapp-dr-bucket` |
| **Region** | Europe — Stockholm (`eu-north-1`) |
| **Lifecycle Rules** | None configured |
| **Replication Rules** | 1 active rule |

**Replication Rule — `crr-rule`**

| Property | Value |
|---|---|
| **Rule Name** | `crr-rule` |
| **Status** | ✅ Enabled |
| **Destination Bucket** | `s3://destiation3` |
| **Destination Region** | Europe (Stockholm) `eu-north-1` |
| **Priority** | 0 |
| **Scope** | Entire bucket |
| **Storage Class** | Same as source |
| **Replica Owner** | Same as source |
| **Replication Time Control (RTC)** | Disabled |
| **KMS Encryption** | Do not replicate |
| **Replica Modification Sync** | Disabled |

---

### Destination Bucket — `destiation3`

| Property | Value |
|---|---|
| **Bucket Name** | `destiation3` |
| **Objects** | 1 |

**Replicated Object:**

| File | Type | Size | Last Modified | Storage Class |
|---|---|---|---|---|
| `Screenshot (28).png` | PNG | 356.3 KB | March 1, 2026 — 13:46:20 (UTC+05:30) | Standard |

This confirms replication is working — the object uploaded to `mapp-dr-bucket` was automatically copied to `destiation3`.

---

### How to Set Up S3 CRR

**Prerequisites:**
- Versioning must be enabled on **both** the source and destination buckets
- An IAM role that allows S3 to replicate on your behalf

**Step 1 — Enable versioning on both buckets:**
```bash
aws s3api put-bucket-versioning \
  --bucket mapp-dr-bucket \
  --versioning-configuration Status=Enabled

aws s3api put-bucket-versioning \
  --bucket destiation3 \
  --versioning-configuration Status=Enabled
```

**Step 2 — Create the replication configuration:**

```json
{
  "Role": "arn:aws:iam::660815084882:role/s3-replication-role",
  "Rules": [
    {
      "ID": "crr-rule",
      "Status": "Enabled",
      "Priority": 0,
      "Filter": {},
      "Destination": {
        "Bucket": "arn:aws:s3:::destiation3",
        "StorageClass": "STANDARD"
      },
      "DeleteMarkerReplication": {
        "Status": "Disabled"
      }
    }
  ]
}
```

```bash
aws s3api put-bucket-replication \
  --bucket mapp-dr-bucket \
  --replication-configuration file://replication.json
```

**Step 3 — Via Console:**
1. Go to **S3 → mapp-dr-bucket → Management tab**
2. Under **Replication rules** → click **Create replication rule**
3. Set rule name: `crr-rule`
4. Source: Entire bucket
5. Destination: `destiation3`
6. IAM Role: Create new or use existing
7. Click **Save**

---

## 🗄️ Part 2 — RDS Read Replica

### What is a Read Replica?

A Read Replica is a live, continuously updated copy of your primary RDS database. It receives all write transactions from the primary in real time. Use cases include offloading read traffic, disaster recovery, and creating a promotion target if the primary fails.

### Primary Database — `database-2`

| Property | Value |
|---|---|
| **DB Identifier** | `database-2` |
| **Role** | Primary |
| **Engine** | MySQL Community |
| **Instance Size** | `db.t3.micro` |
| **Region / AZ** | `eu-north-1b` |
| **Status** | ✅ Available |
| **CPU** | 3.56% |
| **Connections** | 1 |

### Read Replica — `read-replica-db`

| Property | Value |
|---|---|
| **DB Identifier** | `read-replica-db` |
| **Role** | Replica |
| **Engine** | MySQL Community |
| **Instance Size** | `db.t3.micro` |
| **Region / AZ** | `eu-north-1b` |
| **Status** | ✅ Available |
| **Replication State** | 🟢 Replicating |
| **Replication Lag** | **0 Seconds** |
| **Replication Source** | `database-2` |
| **CPU** | 3.04% |
| **Connections** | 0 |
| **Internet Access Gateway** | Disabled |
| **IAM Authentication** | Disabled |

A replication lag of **0 seconds** means the replica is perfectly in sync — any data written to the primary is immediately available on the replica.

---

### Security Group Rules on `read-replica-db`

| Security Group | Type | Rule |
|---|---|---|
| `default` (sg-03707a099a558c793) | EC2 SG — Inbound | sg-03707a099a558c793 |
| `default` (sg-03707a099a558c793) | CIDR/IP — Outbound | `0.0.0.0/0` |
| `ec2-rds-1` (sg-0338f434885257c5c) | EC2 SG — Outbound | sg-0ef795dc4b3eb1c12 |
| `rds-ec2-1` (sg-0ef795dc4b3eb1c12) | CIDR/IP — Inbound | `152.59.121.158/32` |
| `rds-ec2-2` (sg-07498618a89c340d3) | CIDR/IP — Inbound | `152.59.121.158/32` |

---

### How to Create an RDS Read Replica

**Via Console:**
1. Go to **RDS → Databases → database-2**
2. Click **Actions → Create read replica**
3. DB instance identifier: `read-replica-db`
4. Choose same or different region/AZ
5. Instance type: `db.t3.micro`
6. Click **Create read replica**

**Via CLI:**
```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier read-replica-db \
  --source-db-instance-identifier database-2 \
  --db-instance-class db.t3.micro \
  --region eu-north-1
```

---

### Connecting to the Read Replica

The replica exposes its own endpoint. Connect using your MySQL client pointed at the replica's instance endpoint:

```bash
mysql -h read-replica-db.xxxx.eu-north-1.rds.amazonaws.com \
      -u admin \
      -p \
      --ssl-mode=REQUIRED
```

> The replica is **read-only**. Any `INSERT`, `UPDATE`, or `DELETE` statements will fail — only `SELECT` queries are allowed.

---

## ⚡ Failover Playbook

If the primary database (`database-2`) fails:

```
1. Promote the read replica to a standalone primary:
   RDS Console → read-replica-db → Actions → Promote

   Or via CLI:
   aws rds promote-read-replica \
     --db-instance-identifier read-replica-db

2. Update your application's DB connection string to point to:
   read-replica-db.xxxx.eu-north-1.rds.amazonaws.com

3. (Once primary is restored) Create a new read replica from
   the promoted instance to restore your DR setup
```

---

## 🔁 CRR vs RDS Replica — At a Glance

| Feature | S3 CRR | RDS Read Replica |
|---|---|---|
| **Protects** | Object files (S3) | Relational database (RDS) |
| **Replication** | Async, cross-region | Sync/async, same or cross-region |
| **Lag** | Near real-time | 0 seconds (this setup) |
| **Failover** | Manual (switch DNS/app) | Promote replica |
| **Cost** | Data transfer + storage | Instance cost + data transfer |
| **Read offloading** | ❌ Not applicable | ✅ Yes |

---

## 🧹 Cleanup

```bash
# Delete read replica (cannot delete primary while replica exists)
aws rds delete-db-instance \
  --db-instance-identifier read-replica-db \
  --skip-final-snapshot

# Delete primary DB
aws rds delete-db-instance \
  --db-instance-identifier database-2 \
  --skip-final-snapshot

# Remove S3 replication rule (via Console → Management → Delete rule)
# Then empty and delete buckets
aws s3 rm s3://mapp-dr-bucket --recursive
aws s3 rb s3://mapp-dr-bucket

aws s3 rm s3://destiation3 --recursive
aws s3 rb s3://destiation3
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: Europe — Stockholm (`eu-north-1`)