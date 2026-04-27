# 🪣 AWS S3 Bucket — Demo Setup with Bucket Policies

A hands-on AWS S3 project demonstrating how to create an S3 bucket, configure its properties, and apply bucket policies — including a **public read policy** and a **specific IAM user access policy**.

---

## 📌 What This Project Covers

- Creating a general-purpose S3 bucket in AWS
- Understanding bucket properties (versioning, ABAC, tags)
- Writing and applying **bucket policies** in JSON
- Allowing public read access to objects
- Restricting access to a specific IAM user

---

## 🪣 Bucket Details

| Property | Value |
|---|---|
| **Bucket Name** | `prashant-demo-bucket-12345` |
| **ARN** | `arn:aws:s3:::prashant-demo-bucket-12345` |
| **Region** | Asia Pacific — Mumbai (`ap-south-1`) |
| **Created On** | April 7, 2026 at 00:16:08 (UTC+05:30) |
| **Versioning** | Disabled |
| **MFA Delete** | Disabled |
| **ABAC** | Disabled |

---

## 📋 Bucket Policies

S3 bucket policies are written in JSON and control **who can do what** with the objects in your bucket.

> ⚠️ Note: Bucket policies don't apply to objects owned by other AWS accounts.

---

### Policy 1 — Public Read Access

This policy allows **anyone on the internet** to read (download) objects from the bucket. Useful for hosting static websites or public assets.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::prashant-demo-bucket-12345/*"
    }
  ]
}
```

**What each field means:**

| Field | Value | Meaning |
|---|---|---|
| `Sid` | `PublicRead` | A label for this policy statement |
| `Effect` | `Allow` | Grants permission |
| `Principal` | `*` | Anyone — the entire internet |
| `Action` | `s3:GetObject` | Can only **download/read** objects |
| `Resource` | `.../*` | Applies to **all objects** inside the bucket |

> 💡 This does **not** allow uploading, deleting, or listing — only reading.

---

### Policy 2 — Specific IAM User Full Access

This policy grants **full S3 access** to a specific IAM user on both the bucket itself and all its contents. The Principal (IAM user ARN) is redacted in the screenshot.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSpecificUser",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::YOUR_ACCOUNT_ID:user/YOUR_USERNAME"
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::prashant-demo-bucket-12345",
        "arn:aws:s3:::prashant-demo-bucket-12345/*"
      ]
    }
  ]
}
```

**What each field means:**

| Field | Value | Meaning |
|---|---|---|
| `Sid` | `AllowSpecificUser` | Label for this statement |
| `Effect` | `Allow` | Grants permission |
| `Principal` | IAM user ARN | Only **this specific user** |
| `Action` | `s3:*` | **All S3 actions** (read, write, delete, list...) |
| `Resource` | Bucket + `/*` | The bucket itself AND all objects inside it |

> ⚠️ Replace `YOUR_ACCOUNT_ID` and `YOUR_USERNAME` with your actual values before applying.

---

## 🔐 Understanding the Difference

| | Policy 1 (PublicRead) | Policy 2 (AllowSpecificUser) |
|---|---|---|
| **Who** | Everyone | One specific IAM user |
| **What** | Read only (`GetObject`) | Everything (`s3:*`) |
| **Use case** | Static site / public files | Admin or app access |
| **Risk level** | Medium (public exposure) | Low (scoped to one identity) |

---

## 🛠️ How to Apply a Bucket Policy

1. Go to **AWS Console → S3 → Your Bucket**
2. Click the **Permissions** tab
3. Scroll down to **Bucket policy** → click **Edit**
4. Paste your JSON policy
5. Click **Save changes**

Or via AWS CLI:

```bash
aws s3api put-bucket-policy \
  --bucket prashant-demo-bucket-12345 \
  --policy file://bucket-policy.json
```

---

## 📁 Files in This Project

```
.
├── README.md
├── public-read-policy.json        # Policy 1: Open read access
└── specific-user-policy.json      # Policy 2: Scoped IAM user access
```

---

## ⚙️ Bucket Properties Overview

### Versioning
Currently **disabled**. When enabled, S3 keeps a full history of every object — useful if you want to recover accidentally deleted or overwritten files.

### MFA Delete
**Disabled**. When enabled, deleting object versions requires MFA confirmation — adds an extra safety layer for critical data.

### ABAC (Attribute-Based Access Control)
**Disabled**. ABAC lets you write IAM policies based on resource tags rather than explicit resource ARNs — great for scaling access control in large orgs.

---

## 🧹 Cleanup

To avoid storage charges when done:

```bash
# Empty the bucket first
aws s3 rm s3://prashant-demo-bucket-12345 --recursive

# Then delete the bucket
aws s3 rb s3://prashant-demo-bucket-12345
```

> You cannot delete a bucket that still has objects in it.

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1160-3630-7538`  
Region: Asia Pacific — Mumbai (`ap-south-1`)