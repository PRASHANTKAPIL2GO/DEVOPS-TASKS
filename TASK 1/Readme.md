# AWS IAM Best Practices Implementation

## 📌 Project Overview
This project demonstrates the implementation of **AWS Identity and Access Management (IAM) Best Practices** to secure an AWS account using proper user access controls, group-based permissions, Multi-Factor Authentication (MFA), and custom access policies.

The setup follows the **principle of least privilege**, ensuring that users only receive the permissions required for their specific roles.

---

## 🎯 Objectives
- Create IAM groups for role-based access control:
  - **Admin**
  - **Dev**
  - **ReadOnly**
- Assign users to groups based on responsibilities
- Apply least privilege permissions
- Enable MFA for the root account
- Create a custom IAM policy for restricted Amazon S3 folder access

---

## 🛠️ Services Used
- **AWS IAM**
- **AWS S3**
- **AWS Console**

---

## 📂 IAM Structure Implemented

### 👥 IAM Groups
| Group Name | Purpose |
|-----------|---------|
| Admin | Full administrative access |
| Dev | Limited developer permissions |
| ReadOnly | Read-only access to AWS resources |

---

### 👤 IAM Users
| User Name | Assigned Group |
|----------|---------------|
| admin-user | Admin |
| dev-user | Dev |
| read-user | ReadOnly |

---

## 🔐 Security Best Practices Applied

### ✅ Root Account Protection
- Enabled **Multi-Factor Authentication (MFA)** on root account
- Removed active root access keys

### ✅ Least Privilege Access
Each IAM group was assigned only the permissions necessary for its tasks.

Examples:

#### Admin Group
- Full AWS administrative access

#### Dev Group
- Access to development-related services only
- Restricted from billing and IAM modifications

#### ReadOnly Group
- Read-only access across AWS resources

---

## 📁 Custom IAM Policy – Restricted S3 Folder Access

A custom IAM policy was created to allow users access only to a specific folder inside an S3 bucket.
