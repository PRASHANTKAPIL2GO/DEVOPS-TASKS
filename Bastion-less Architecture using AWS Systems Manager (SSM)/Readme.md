# 🔒 AWS Private VPC with SSM Access — No SSH, No Internet

A secure AWS networking setup that lets you connect to a **fully private EC2 instance** (no public IP, no internet gateway) using **AWS Systems Manager (SSM)** via VPC Interface Endpoints — no bastion hosts, no SSH keys needed.

---

## 📌 What This Project Does

- Creates a **private VPC** with no internet access
- Launches an EC2 instance with **no public IP**
- Connects to that instance securely using **SSM Session Manager** through VPC endpoints
- Uses an **IAM role** to grant the EC2 instance permission to talk to SSM services
- Demonstrates how to reach private AWS services (SSM, EC2 Messages, S3) without leaving the AWS network

---

## 🏗️ Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                  private-access-vpc (10.0.0.0/16)        │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │           private-subnet (10.0.1.0/24)              │ │
│  │                                                     │ │
│  │   ┌──────────────────────────────┐                  │ │
│  │   │  Private-SSM-EC2             │                  │ │
│  │   │  Instance: i-056c0fb89f0130cc2                  │ │
│  │   │  Private IP: 10.0.1.189      │                  │ │
│  │   │  No public IP ❌             │                  │ │
│  │   │  IAM Role: SSM-Private-EC2   │                  │ │
│  │   └──────────────────────────────┘                  │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  VPC Interface Endpoints (PrivateLink):                  │
│  ├── my-endpoint-1  → ec2messages  (SSM agent comms)     │
│  ├── my-endpoint-2  → ssmmessages  (session data)        │
│  ├── my-endpoint-3  → ssm          (SSM core service)    │
│  └── s3-endpoint    → s3           (S3 access)           │
│                                                          │
│  Route Table: Private-rt                                 │
│  └── 10.0.0.0/16 → local only (no internet route)       │
└──────────────────────────────────────────────────────────┘
          │
          │  SSM Session Manager (via HTTPS/443)
          │
     AWS Console / CLI (your computer)
```

> 🔑 **Key insight**: Instead of routing traffic to the internet to reach SSM, the VPC endpoints bring SSM *into* your VPC so the EC2 instance never needs to leave.

---

## 🧩 AWS Resources

### VPC

| Property | Value |
|---|---|
| **Name** | `private-access-vpc` |
| **VPC ID** | `vpc-0578f3bcd91ce502f` |
| **CIDR** | `10.0.0.0/16` |
| **Region** | Asia Pacific — Mumbai (`ap-south-1`) |
| **Block Public Access** | Off |

---

### Private Subnet

| Property | Value |
|---|---|
| **Name** | `private-subnet` |
| **Subnet ID** | `subnet-0528d821b75271674` |
| **CIDR** | `10.0.1.0/24` |
| **Availability Zone** | `eu-north-1c` (eun1-az3) |
| **Available IPs** | 246 |
| **Auto-assign Public IP** | ❌ No |
| **Default subnet** | ❌ No |
| **Route Table** | `Private-rt` |

---

### Route Table — Private-rt

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | `local` | Internal VPC traffic only |

> There is **no `0.0.0.0/0` route** — meaning no internet access. All traffic stays inside the VPC.

---

### EC2 Instance — Private-SSM-EC2

| Property | Value |
|---|---|
| **Instance ID** | `i-056c0fb89f0130cc2` |
| **Instance Type** | `t3.micro` |
| **Private IP** | `10.0.1.189` |
| **Public IP** | ❌ None |
| **AMI** | `al2023-ami-2023.10.20260216.1-kernel-6.1-x86_64` |
| **IAM Role** | `SSM-Private-EC2-Role` |
| **VPC** | `private-access-vpc` |
| **Subnet** | `private-subnet` |
| **IMDSv2** | Required |
| **Termination Protection** | Disabled |

---

### Security Group — Private-EC2-SG

| Property | Value |
|---|---|
| **Name** | `Private-EC2-SG` |
| **SG ID** | `sg-0f63838bf60060f8e` |
| **Description** | `private sg` |
| **VPC** | `vpc-0578f3bcd91ce502f` |

**Inbound Rules:**

| Type | Protocol | Port | Source | Why |
|---|---|---|---|---|
| HTTPS | TCP | 443 | `10.0.0.0/16` | Allows SSM agent to communicate via VPC endpoints |

> No port 22 (SSH) — access is entirely through SSM.

---

### IAM Role — SSM-Private-EC2-Role

| Property | Value |
|---|---|
| **Role Name** | `SSM-Private-EC2-Role` |
| **ARN** | `arn:aws:iam::660815084882:role/SSM-Private-EC2-Role` |
| **Created** | March 01, 2026 |
| **Last Activity** | 1 month ago |
| **Max Session** | 1 hour |
| **Policy Attached** | `AmazonSSMManagedInstanceCore` (AWS managed) |

This role allows the EC2 instance to register with SSM and accept session connections — without any human credentials.

---

### VPC Interface Endpoints

These endpoints let the private EC2 instance talk to AWS services *without going through the internet*:

| Name | Service | Type | Status |
|---|---|---|---|
| `my-endpoint-1` | `com.amazonaws.eu-north-1.ec2messages` | Interface | ✅ Available |
| `my-endpoint-2` | `com.amazonaws.eu-north-1.ssmmessages` | Interface | ✅ Available |
| `my-endpoint-3` | `com.amazonaws.eu-north-1.ssm` | Interface | ✅ Available |
| `s3-endpoint` | `com.amazonaws.eu-north-1.s3` | Interface | ✅ Available |

> These 3 SSM endpoints (`ssm`, `ssmmessages`, `ec2messages`) are the **minimum required** for SSM Session Manager to work in a private subnet.

---

## ⚙️ Setup Guide

### Step 1 — Create the VPC

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region eu-north-1
# Tag it: private-access-vpc
```

### Step 2 — Create a Private Subnet

```bash
aws ec2 create-subnet \
  --vpc-id vpc-0578f3bcd91ce502f \
  --cidr-block 10.0.1.0/24 \
  --availability-zone eu-north-1c
# Tag it: private-subnet
# Do NOT enable auto-assign public IP
```

### Step 3 — Create a Private Route Table

```bash
aws ec2 create-route-table --vpc-id vpc-0578f3bcd91ce502f
# Associate it with private-subnet
# Add NO internet route (0.0.0.0/0) — keep it local only
```

### Step 4 — Create IAM Role for SSM

1. Go to **IAM → Roles → Create role**
2. Trusted entity: **EC2**
3. Attach policy: `AmazonSSMManagedInstanceCore`
4. Name it: `SSM-Private-EC2-Role`

### Step 5 — Create VPC Interface Endpoints

For each of the 3 SSM services (+ S3 if needed):

1. Go to **VPC → Endpoints → Create endpoint**
2. Select type: **Interface**
3. Choose service: `com.amazonaws.eu-north-1.ssm`
4. Select your VPC and private subnet
5. Enable **Private DNS names**
6. Repeat for `ssmmessages` and `ec2messages`

### Step 6 — Create Security Group

Allow inbound HTTPS (port 443) from within the VPC (`10.0.0.0/16`):

```bash
aws ec2 create-security-group \
  --group-name Private-EC2-SG \
  --description "private sg" \
  --vpc-id vpc-0578f3bcd91ce502f

aws ec2 authorize-security-group-ingress \
  --group-id sg-0f63838bf60060f8e \
  --protocol tcp --port 443 \
  --cidr 10.0.0.0/16
```

### Step 7 — Launch the EC2 Instance

1. Go to **EC2 → Launch Instance**
2. Choose AMI: Amazon Linux 2023
3. Instance type: `t3.micro`
4. **No key pair** (you won't need SSH)
5. Network: `private-access-vpc` → `private-subnet`
6. **Disable** auto-assign public IP
7. IAM instance profile: `SSM-Private-EC2-Role`
8. Security group: `Private-EC2-SG`

---

## 🖥️ Connecting via SSM Session Manager

Once the instance is running, connect directly from the AWS Console:

1. Go to **EC2 → Instances → Select your instance**
2. Click **Connect → Session Manager → Connect**

Or via CLI:

```bash
aws ssm start-session --target i-056c0fb89f0130cc2 --region eu-north-1
```

Once connected, you'll be in as `ssm-user`:

```bash
sh-5.2$ whoami
ssm-user

sh-5.2$ ip route
default via 10.0.1.1 dev ens5 proto dhcp src 10.0.1.189 metric 512
10.0.0.2 via 10.0.1.1 dev ens5 proto dhcp src 10.0.1.189 metric 512
10.0.1.0/24 dev ens5 proto kernel scope link src 10.0.1.189
```

> ✅ The instance has a private IP, no public route, and is reachable only through SSM.

---

## 🔐 Why This Setup is More Secure Than SSH

| | Traditional SSH | This Setup (SSM) |
|---|---|---|
| **Public IP needed** | ✅ Yes | ❌ No |
| **Port 22 open** | ✅ Yes | ❌ No |
| **Key pair management** | Required | Not needed |
| **Bastion host** | Often needed | Not needed |
| **Audit trail** | Manual | Built-in (CloudTrail) |
| **Access control** | Key-based | IAM policies |

---

## 🧹 Cleanup

To avoid charges, delete resources in this order:

```
1. Terminate the EC2 instance
2. Delete VPC Endpoints (they have an hourly cost)
3. Delete the Security Group
4. Delete the Subnet
5. Delete the Route Table
6. Delete the VPC
7. Delete the IAM Role (optional — no cost)
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: Europe — Stockholm (`eu-north-1`)