# AWS VPC Endpoint for Amazon S3 (Private Access)

## 📌 Project Overview

This project demonstrates the creation and configuration of an **AWS VPC Interface Endpoint** for **Amazon S3** inside a custom VPC.  
The endpoint enables secure and private communication between resources in the VPC and Amazon S3 **without using the public internet, NAT Gateway, or Internet Gateway**.

This setup improves:

- Security 🔒
- Network efficiency ⚡
- Lower data transfer exposure 🌐
- Private AWS service connectivity ☁️

---

## 🏗️ Architecture Used

- **VPC Name:** hub-vpc  
- **VPC ID:** `vpc-06df5f2a33660fdeb`
- **Service:** Amazon S3
- **Endpoint Type:** Interface Endpoint
- **Region:** eu-north-1
- **Subnet Attached:** hub subnet
- **Availability Zone:** eu-north-1a

---

## 📷 Proof of Deployment

The endpoint was successfully created and is in **Available** state.

### Endpoint Details:

| Property | Value |
|--------|-------|
| Endpoint ID | vpce-082f63fd4a7a55a4a |
| Service Name | com.amazonaws.eu-north-1.s3 |
| Endpoint Type | Interface |
| IP Address Type | IPv4 |
| Status | Available |
| Private DNS Enabled | No |

---

## 🚀 Implementation Steps

### Step 1: Open AWS Console

Navigate to:

```text
VPC Dashboard → Endpoints