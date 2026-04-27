# 💾 AWS EBS — Volumes, Snapshots & Data Recovery

A hands-on AWS storage project demonstrating how to create EBS volumes, take snapshots, restore data from a snapshot to a new volume, attach it to an EC2 instance, and access the recovered files — all without losing a byte.

---

## 📌 What This Project Does

- Creates and manages **EBS volumes** (gp2 and gp3) on EC2
- Takes an **EBS snapshot** as a point-in-time backup
- Restores data by creating a **new volume from a snapshot** and attaching it to an instance
- Mounts the restored volume and **verifies the data is intact**
- Demonstrates the full backup → restore → verify cycle

---

## 🏗️ How It All Fits Together

```
Original EC2 Instance
        │
        │ (has data in /home/ubuntu/mydata/)
        │   file1.txt → "Hello AWS Snapshot"
        │   file2.txt
        ▼
  EBS Volume (xvda - root, 8 GiB)
        │
        │ Take Snapshot
        ▼
  snap-08476ee1bb85ab8d8
  (snapshot-test | 8 GiB | Completed)
        │
        │ Create new volume from snapshot
        ▼
  vol-0eefa40e57055b8b6 (gp3, 8 GiB)
        │
        │ Attach to EC2 instance i-0638074601898df3f
        ▼
  Appears as /dev/xvdf on the instance
        │
        │ Mount: sudo mount /dev/xvdf1 /mnt/data
        ▼
  /mnt/data/home/ubuntu/mydata/
        ├── file1.txt  → "Hello AWS Snapshot" ✅
        └── file2.txt                          ✅
```

---

## 🧩 EBS Volumes

| Volume ID | Type | Size | IOPS | Throughput | Snapshot Source |
|---|---|---|---|---|---|
| `vol-05bce723b8f07a377` | gp2 | 8 GiB | 100 | — | `snap-087ee8e...` |
| `vol-0eefa40e57055b8b6` | gp3 | 8 GiB | 3000 | 125 MB/s | `snap-08476ee...` |
| `vol-0fd6f0a5e408eb7dd` | gp3 | 8 GiB | 3000 | 125 MB/s | `snap-08cb0aa...` |

> ✅ `vol-0eefa40e57055b8b6` was successfully attached to instance `i-0638074601898df3f`

**Snapshot summary (as of Apr 03, 2026):** 1 out of 3 volumes has been backed up.

> ⚠️ No Data Lifecycle Manager (DLM) policy is set up — snapshots are currently manual. Consider automating this for production.

---

## 📸 Snapshot

| Property | Value |
|---|---|
| **Snapshot ID** | `snap-08476ee1bb85ab8d8` |
| **Description** | `snapshot-test` |
| **Full Snapshot Size** | 8 GiB |
| **Volume Size** | 8 GiB |
| **Storage Tier** | Standard |
| **Status** | ✅ Completed |

---

## 🖥️ Block Device Layout (on the EC2 instance)

Output of `lsblk` before and after attaching the restored volume:

```
NAME      SIZE  TYPE  MOUNTPOINTS
loop0    25.2M  loop  /snap/amazon-ssm-agent/7983
loop1    55.7M  loop  /snap/core18/2812
loop2    38.7M  loop  /snap/snapd/21465

xvda      8G    disk                      ← Root volume (OS)
├─xvda1   7G    part  /                   ← Root partition
├─xvda14  4M    part
├─xvda15  106M  part  /boot/efi
└─xvda16  913M  part  /boot

xvdf      8G    disk                      ← Restored volume (from snapshot)
├─xvdf1   7G    part                      ← Mount this partition!
├─xvdf14  4M    part
├─xvdf15  106M  part
└─xvdf16  913M  part
```

---

## ⚙️ Step-by-Step Guide

### Step 1 — Create a Snapshot of Your Volume

In the console:
1. Go to **EC2 → Elastic Block Store → Volumes**
2. Select your volume → **Actions → Create snapshot**
3. Add description: `snapshot-test`
4. Click **Create snapshot**

Or via CLI:
```bash
aws ec2 create-snapshot \
  --volume-id vol-05bce723b8f07a377 \
  --description "snapshot-test" \
  --region ap-south-1
```

Wait for status to become **Completed**.

---

### Step 2 — Create a New Volume from the Snapshot

```bash
aws ec2 create-volume \
  --snapshot-id snap-08476ee1bb85ab8d8 \
  --volume-type gp3 \
  --availability-zone ap-south-1a \
  --region ap-south-1
```

> Make sure the new volume is in the **same Availability Zone** as your EC2 instance — EBS volumes can only attach to instances in the same AZ.

---

### Step 3 — Attach the Volume to Your EC2 Instance

```bash
aws ec2 attach-volume \
  --volume-id vol-0eefa40e57055b8b6 \
  --instance-id i-0638074601898df3f \
  --device /dev/xvdf \
  --region ap-south-1
```

You'll see a green success banner in the console:
> ✅ Successfully attached volume `vol-0eefa40e57055b8b6` to instance `i-0638074601898df3f`

---

### Step 4 — Check the Disk Type Before Mounting

When you first try to mount the whole disk (`/dev/xvdf`), you'll hit this error:

```bash
$ sudo mount /dev/xvdf /mnt/data
mount: /mnt/data: wrong fs type, bad option, bad superblock on /dev/xvdf...
```

This happens because the snapshot was taken from a **partitioned disk** — you need to mount the **partition** (`/dev/xvdf1`), not the raw disk.

Confirm this with:
```bash
$ sudo file -s /dev/xvdf
/dev/xvdf: DOS/MBR boot sector, extended partition table (last)
```

This tells you the disk has a partition table — so always mount `xvdf1`, not `xvdf`.

---

### Step 5 — Mount the Correct Partition

```bash
# Create mount point
sudo mkdir /mnt/data

# Mount the partition (not the disk!)
sudo mount /dev/xvdf1 /mnt/data

# Verify it mounted
cd /mnt/data
ls
# bin  boot  etc  home  lib  lib64  media  mnt  opt  proc  root  sbin  snap  srv  sys  usr  var
```

---

### Step 6 — Verify Your Data

Navigate to where the original data was stored:

```bash
cd /mnt/data/home/ubuntu/mydata
ls
# file1.txt  file2.txt

cat file1.txt
# Hello AWS Snapshot
```

✅ Data successfully recovered from the snapshot!

---

## 🔁 gp2 vs gp3 — Which Should You Use?

| Feature | gp2 | gp3 |
|---|---|---|
| IOPS | Tied to size (3 IOPS/GiB) | Fixed 3,000 baseline |
| Throughput | Up to 250 MB/s | Up to 1,000 MB/s |
| Cost | Standard | ~20% cheaper than gp2 |
| Best for | Legacy workloads | Most new workloads |

> This project uses **gp3** for the restored volumes — better performance at lower cost.

---

## ⚠️ Common Mistakes to Avoid

| Mistake | What Happens | Fix |
|---|---|---|
| Mounting `/dev/xvdf` instead of `/dev/xvdf1` | Mount fails with superblock error | Run `lsblk` first, always mount the partition |
| Wrong Availability Zone | Volume attachment fails | EBS must be in same AZ as EC2 |
| No snapshot before terminating | Data lost permanently | Always snapshot before stopping/terminating |
| Forgetting to unmount before detach | Risk of data corruption | `sudo umount /mnt/data` first |

---

## 🔄 Automating Snapshots with Data Lifecycle Manager

Currently snapshots in this project are **manual**. For production, set up automatic snapshots:

1. Go to **EC2 → Elastic Block Store → Lifecycle Manager**
2. Click **Create lifecycle policy**
3. Set schedule: e.g., daily at 00:00 UTC, retain 7 snapshots
4. Apply to volumes by tag (e.g., `Backup=true`)

Or via CLI:
```bash
aws dlm create-lifecycle-policy \
  --description "Daily EBS snapshots" \
  --state ENABLED \
  --execution-role-arn arn:aws:iam::YOUR_ACCOUNT:role/AWSDataLifecycleManagerDefaultRole \
  --policy-details file://dlm-policy.json
```

---

## 🧹 Cleanup

```bash
# 1. Unmount the volume first
sudo umount /mnt/data

# 2. Detach the volume from the instance
aws ec2 detach-volume --volume-id vol-0eefa40e57055b8b6

# 3. Delete volumes you no longer need
aws ec2 delete-volume --volume-id vol-0eefa40e57055b8b6

# 4. Delete the snapshot
aws ec2 delete-snapshot --snapshot-id snap-08476ee1bb85ab8d8
```

> Delete snapshots only after confirming you no longer need the backup. Once deleted, they cannot be recovered.

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1160-3630-7538`  
Region: Asia Pacific — Mumbai (`ap-south-1`)