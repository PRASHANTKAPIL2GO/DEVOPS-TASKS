# 🔐 AWS IAM — Users, Groups & Access Management

A structured AWS Identity and Access Management (IAM) setup that organizes users into permission-based groups, follows security best practices, and demonstrates the right way to manage access in an AWS account — without using the root user for day-to-day tasks.

---

## 📌 What This Project Does

- Creates **IAM users** with distinct roles: admin, developer, and read-only
- Organizes users into **IAM groups** with appropriate permissions attached
- Ensures the **root account is secured** with MFA and no active access keys
- Sets up **18 IAM roles** for service-to-service access (EC2, Lambda, etc.)
- Applies **7 custom/managed policies** across the account

---

## 🔒 Security Posture — Dashboard Check

The IAM dashboard shows **0 open security recommendations** — both critical checks are green:

| Check | Status | Why It Matters |
|---|---|---|
| Root user has MFA | ✅ Enabled | Protects the most powerful account from takeover |
| Root user has no active access keys | ✅ None | Root keys are a major security risk — using IAM users instead is the right approach |

> 🏆 A clean IAM dashboard with 0 security recommendations is the goal for every AWS account.

---

## 📊 IAM Resources Summary

| Resource | Count |
|---|---|
| User Groups | 3 |
| Users | 4 |
| Roles | 18 |
| Policies | 7 |
| Identity Providers | 0 |

**Account ID:** `1140-1518-7882`  
**Sign-in URL:** `https://660815084882.signin.aws.amazon.com/console`

---

## 👥 IAM Users

| Username | Groups | Last Activity | MFA | Access Key | Key Age |
|---|---|---|---|---|---|
| `admin-user` | 1 | ✅ 7 days ago | — | Active (`AKIAZTW5F7V...`) | 7 days |
| `dev-user` | 1 | — | — | — | — |
| `read-user` | 1 | — | — | — | — |
| *(4th user)* | — | — | — | — | — |

> ⚠️ **Security note**: `admin-user` has an active access key that is 7 days old. Access keys should be rotated regularly (best practice: every 90 days). Also consider enabling MFA on all users, especially admin-user.

---

## 🗂️ IAM User Groups

| Group | Users | Permissions | Created |
|---|---|---|---|
| `Admin` | 2 | ✅ Defined | 1 month ago |
| `Dev` | 1 | ✅ Defined | 1 month ago |
| `ReadOnly` | 1 | ✅ Defined | 1 month ago |

### Group Design

```
Admin Group (2 users)
└── admin-user + 1 more
└── Policy: AdministratorAccess (or similar full-access policy)
└── Can: manage all AWS resources

Dev Group (1 user)
└── dev-user
└── Policy: Developer-scoped permissions (EC2, S3, RDS, Lambda, etc.)
└── Can: build and deploy, cannot manage IAM or billing

ReadOnly Group (1 user)
└── read-user
└── Policy: ReadOnlyAccess (AWS managed)
└── Can: view all resources, cannot make any changes
```

---

## ⚙️ Setup Guide

### Step 1 — Secure the Root Account

Before creating any IAM resources, secure root:

1. Log in as root → **My Security Credentials**
2. Enable **MFA** (virtual MFA via Authenticator app)
3. Ensure **no access keys** exist for root (delete if any)
4. Set a **strong password** and store it securely

---

### Step 2 — Create User Groups with Policies

**Admin group:**
```bash
aws iam create-group --group-name Admin

aws iam attach-group-policy \
  --group-name Admin \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

**Dev group:**
```bash
aws iam create-group --group-name Dev

aws iam attach-group-policy \
  --group-name Dev \
  --policy-arn arn:aws:iam::aws:policy/PowerUserAccess
```

**ReadOnly group:**
```bash
aws iam create-group --group-name ReadOnly

aws iam attach-group-policy \
  --group-name ReadOnly \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

---

### Step 3 — Create IAM Users

```bash
# Create users
aws iam create-user --user-name admin-user
aws iam create-user --user-name dev-user
aws iam create-user --user-name read-user
```

Enable console access for each:
```bash
aws iam create-login-profile \
  --user-name admin-user \
  --password "TempPassword123!" \
  --password-reset-required
```

---

### Step 4 — Add Users to Groups

```bash
aws iam add-user-to-group --user-name admin-user --group-name Admin
aws iam add-user-to-group --user-name dev-user --group-name Dev
aws iam add-user-to-group --user-name read-user --group-name ReadOnly
```

---

### Step 5 — Create Access Keys (for admin-user)

```bash
aws iam create-access-key --user-name admin-user
```

> Save the `AccessKeyId` and `SecretAccessKey` immediately — the secret is only shown once.

Configure it locally:
```bash
aws configure --profile admin
# AWS Access Key ID: AKIAZTW5F7V...
# AWS Secret Access Key: <secret>
# Default region: eu-north-1
# Default output format: json
```

---

## 🎭 IAM Roles (18 total)

The account has 18 IAM roles — these are used for **service-to-service permissions**, such as:

| Role Type | Example | Used By |
|---|---|---|
| EC2 instance roles | `ec2-s3-role`, `SSM-Private-EC2-Role` | EC2 instances accessing S3, SSM |
| Lambda execution roles | Allow Lambda to write logs, access DynamoDB | Lambda functions |
| Service-linked roles | Auto-created by AWS services | ELB, Auto Scaling, RDS |
| Cross-account roles | Allow other accounts to assume access | Multi-account setups |

> Roles use **temporary credentials** — far more secure than long-lived access keys.

---

## 📜 Policies Overview (7 total)

Policies define what actions are allowed or denied. This account has 7 policies across:

| Policy Type | Examples |
|---|---|
| AWS Managed | `AdministratorAccess`, `ReadOnlyAccess`, `AmazonSSMManagedInstanceCore` |
| Custom | Scoped policies for specific services or teams |

**Example custom policy — S3 read-only for a specific bucket:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

---

## 🏛️ IAM Best Practices Applied

| Practice | Status in This Setup |
|---|---|
| Root MFA enabled | ✅ Done |
| No root access keys | ✅ Done |
| Users grouped by role | ✅ Admin, Dev, ReadOnly |
| Least privilege principle | ✅ ReadOnly and Dev have scoped access |
| No direct policy on users | ✅ Permissions via groups only |
| Roles for EC2/services | ✅ 18 roles configured |
| Access key rotation | ⚠️ admin-user key is 7 days old — set a reminder to rotate at 90 days |
| MFA on IAM users | ⚠️ Not enabled on individual users — recommended for admin-user |

---

## 🔁 IAM vs Root User — When to Use What

| Task | Use |
|---|---|
| Creating first IAM user | Root (one time only) |
| Day-to-day AWS work | IAM user or role |
| Billing and account settings | Root |
| Deploying applications | IAM role (on EC2/Lambda) |
| Emergency account recovery | Root |
| Everything else | IAM user with least privilege |

> **Golden rule**: Log in as root only when absolutely necessary. Use `admin-user` for everything else.

---

## 🧹 Cleanup

```bash
# Remove users from groups first
aws iam remove-user-from-group --user-name admin-user --group-name Admin
aws iam remove-user-from-group --user-name dev-user --group-name Dev
aws iam remove-user-from-group --user-name read-user --group-name ReadOnly

# Delete login profiles
aws iam delete-login-profile --user-name admin-user

# Delete access keys
aws iam delete-access-key \
  --user-name admin-user \
  --access-key-id AKIAZTW5F7V...

# Delete users
aws iam delete-user --user-name admin-user
aws iam delete-user --user-name dev-user
aws iam delete-user --user-name read-user

# Detach policies and delete groups
aws iam detach-group-policy --group-name Admin \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam delete-group --group-name Admin
# Repeat for Dev and ReadOnly
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
IAM Sign-in URL: `https://660815084882.signin.aws.amazon.com/console`