# 🌐 AWS Hub-and-Spoke VPC Peering

A multi-VPC networking setup using **VPC Peering** in a hub-and-spoke topology — where a central **hub VPC** connects to multiple **spoke VPCs**, allowing traffic to flow between them without going through the internet.

---

## 📌 What This Project Does

- Creates **3 VPCs** — one hub and two spokes — each with non-overlapping CIDR blocks
- Establishes **VPC Peering connections** between the hub and each spoke
- Configures **route tables** in each VPC so they can reach each other
- Demonstrates how to build private, inter-VPC communication entirely within AWS

---

## 🏗️ Architecture Overview

```
                    ┌─────────────────────┐
                    │      hub-vpc         │
                    │  10.0.0.0/16        │
                    │  vpc-06df5f2a33660fdeb│
                    │                     │
                    │  Route Table:        │
                    │  10.1.0.0/16 → pcx-01fb6ff5a2cbda9f6 (spoke1)
                    │  10.2.0.0/16 → pcx-0b612c844b3b691d4 (spoke2)
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
     VPC Peering                        VPC Peering
  pcx-01fb6ff5a2cbda9f6             pcx-0b612c844b3b691d4
              │                                 │
   ┌──────────▼──────────┐         ┌────────────▼────────┐
   │      spoke1-vpc      │         │      spoke2-vpc      │
   │   10.1.0.0/16       │         │   10.2.0.0/16       │
   │  vpc-09c8c42ed0a2f23f5        │  vpc-069b8f302dad3861b│
   │                     │         │                     │
   │  Route Table:        │         │  Route Table:        │
   │  10.0.0.0/16 → pcx  │         │  10.0.0.0/16 → pcx  │
   └─────────────────────┘         └─────────────────────┘
```

> ⚠️ **Important**: Spoke VPCs do NOT peer with each other directly. Traffic between spoke1 and spoke2 is not possible in this setup without transitive routing (which VPC peering doesn't support natively — use Transit Gateway for that).

---

## 🧩 VPC Inventory

| Name | VPC ID | CIDR | Role |
|---|---|---|---|
| `hub-vpc` | `vpc-06df5f2a33660fdeb` | `10.0.0.0/16` | Central hub |
| `spoke1-vpc` | `vpc-09c8c42ed0a2f23f5` | `10.1.0.0/16` | Spoke 1 |
| `spoke2-vpc` | `vpc-069b8f302dad3861b` | `10.2.0.0/16` | Spoke 2 |
| `private-access-vpc` | `vpc-0578f3bcd91ce502f` | `10.0.0.0/16` | Separate project |
| `new-vpc` | `vpc-03434cafac89b43b1` | `172.31.0.0/16` | Default/other |

> All VPCs are in **Europe (Stockholm) — `eu-north-1`** and owned by account `660815084882`.

---

## 🔗 VPC Peering Connections

| Peering ID | Between | Direction |
|---|---|---|
| `pcx-01fb6ff5a2cbda9f6` | hub-vpc ↔ spoke1-vpc | Bidirectional |
| `pcx-0b612c844b3b691d4` | hub-vpc ↔ spoke2-vpc | Bidirectional |

---

## 🗺️ Route Tables

Route tables are what actually make peering work — creating a peering connection alone isn't enough. You must add routes in **both** VPCs pointing to the peering connection.

### hub-vpc Route Table (`rtb-02a632cbf77eb42b9`)

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | `local` | Traffic within hub-vpc |
| `10.1.0.0/16` | `pcx-01fb6ff5a2cbda9f6` | Route to spoke1-vpc |
| `10.2.0.0/16` | `pcx-0b612c844b3b691d4` | Route to spoke2-vpc |

The hub knows how to reach **both** spokes. This is the "center" of the topology.

---

### spoke1-vpc Route Table (`rtb-0a75b9785cfc62f43`)

| Destination | Target | Purpose |
|---|---|---|
| `10.1.0.0/16` | `local` | Traffic within spoke1-vpc |
| `10.0.0.0/16` | `pcx-01fb6ff5a2cbda9f6` | Route back to hub-vpc |

Spoke1 can reach the hub, but **not** spoke2 directly.

---

### spoke2-vpc Route Table (`rtb-0348d2e9c67c73766`)

| Destination | Target | Purpose |
|---|---|---|
| `10.2.0.0/16` | `local` | Traffic within spoke2-vpc |
| `10.0.0.0/16` | `pcx-0b612c844b3b691d4` | Route back to hub-vpc |

Spoke2 can reach the hub, but **not** spoke1 directly.

---

## ⚙️ Setup Guide

### Step 1 — Create the VPCs

Make sure your CIDRs **don't overlap** — peering fails if they do.

```bash
# Hub VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region eu-north-1
# Tag: hub-vpc

# Spoke 1
aws ec2 create-vpc --cidr-block 10.1.0.0/16 --region eu-north-1
# Tag: spoke1-vpc

# Spoke 2
aws ec2 create-vpc --cidr-block 10.2.0.0/16 --region eu-north-1
# Tag: spoke2-vpc
```

### Step 2 — Create VPC Peering Connections

```bash
# Hub ↔ Spoke1
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-06df5f2a33660fdeb \
  --peer-vpc-id vpc-09c8c42ed0a2f23f5 \
  --region eu-north-1

# Hub ↔ Spoke2
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-06df5f2a33660fdeb \
  --peer-vpc-id vpc-069b8f302dad3861b \
  --region eu-north-1
```

Then **accept** each peering request:

```bash
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-01fb6ff5a2cbda9f6

aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-0b612c844b3b691d4
```

### Step 3 — Update Route Tables

Add routes in **all three** route tables. Without this step, peering exists but traffic won't flow.

**In hub-vpc route table:**
```bash
# Route to spoke1
aws ec2 create-route \
  --route-table-id rtb-02a632cbf77eb42b9 \
  --destination-cidr-block 10.1.0.0/16 \
  --vpc-peering-connection-id pcx-01fb6ff5a2cbda9f6

# Route to spoke2
aws ec2 create-route \
  --route-table-id rtb-02a632cbf77eb42b9 \
  --destination-cidr-block 10.2.0.0/16 \
  --vpc-peering-connection-id pcx-0b612c844b3b691d4
```

**In spoke1-vpc route table:**
```bash
aws ec2 create-route \
  --route-table-id rtb-0a75b9785cfc62f43 \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id pcx-01fb6ff5a2cbda9f6
```

**In spoke2-vpc route table:**
```bash
aws ec2 create-route \
  --route-table-id rtb-0348d2e9c67c73766 \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id pcx-0b612c844b3b691d4
```

### Step 4 — Update Security Groups

Make sure instances in each VPC allow inbound traffic from the peered CIDR ranges. For example, to allow ICMP (ping) from hub-vpc into spoke1:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <spoke1-sg-id> \
  --protocol icmp \
  --port -1 \
  --cidr 10.0.0.0/16
```

---

## ✅ Verify Connectivity

SSH or SSM into an EC2 instance in one VPC and ping an instance in another:

```bash
# From hub-vpc instance (10.0.x.x), ping spoke1 instance
ping 10.1.0.5

# From hub-vpc instance, ping spoke2 instance
ping 10.2.0.8
```

If routes and security groups are configured correctly, these should succeed.

---

## ⚠️ VPC Peering Limitations

| Limitation | What it means |
|---|---|
| **No transitive routing** | spoke1 cannot talk to spoke2 through the hub |
| **No overlapping CIDRs** | All VPC CIDRs must be unique |
| **Same or different accounts** | Works cross-account, but requires accept step |
| **No edge-to-edge routing** | Can't use peering to route through NAT/IGW of another VPC |

> 💡 If you need spoke-to-spoke communication or more complex routing, consider upgrading to **AWS Transit Gateway**.

---

## 🔁 Hub-and-Spoke vs Transit Gateway

| Feature | VPC Peering (this setup) | Transit Gateway |
|---|---|---|
| Cost | Free (data transfer costs apply) | Per attachment + data |
| Spoke-to-spoke | ❌ Not supported | ✅ Supported |
| Scalability | Harder (N² connections) | Easy (star topology) |
| Setup complexity | Low | Medium |
| Best for | Small setups (2-5 VPCs) | Large, growing networks |

---

## 🧹 Cleanup

```bash
# 1. Delete routes pointing to peering connections (in each route table)
# 2. Delete peering connections
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-01fb6ff5a2cbda9f6
aws ec2 delete-vpc-peering-connection --vpc-peering-connection-id pcx-0b612c844b3b691d4

# 3. Terminate any EC2 instances in these VPCs
# 4. Delete subnets, then VPCs
aws ec2 delete-vpc --vpc-id vpc-09c8c42ed0a2f23f5
aws ec2 delete-vpc --vpc-id vpc-069b8f302dad3861b
aws ec2 delete-vpc --vpc-id vpc-06df5f2a33660fdeb
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: Europe — Stockholm (`eu-north-1`)