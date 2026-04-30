# Prashant-VPC — AWS VPC Networking Setup

## Overview

`Prashant-VPC` is a custom Amazon VPC deployed in the **Asia Pacific (Tokyo)** region (`ap-northeast-1`), managed under the AWS account **MANAS (660815084882)** via the `terraform-prashant` IAM role. It follows a standard public/private subnet architecture with internet access via an Internet Gateway and outbound-only access from private resources via a NAT Gateway.

---

## VPC Details

| Property               | Value                              |
|------------------------|------------------------------------|
| **VPC Name**           | Prashant-VPC                       |
| **VPC ID**             | `vpc-014b288abcb6e1b05`            |
| **Region**             | Asia Pacific (Tokyo) — `ap-northeast-1` |
| **IPv4 CIDR**          | `10.0.0.0/16`                      |
| **State**              | Available                          |
| **Default VPC**        | No                                 |
| **Tenancy**            | Default                            |
| **DNS Resolution**     | Enabled                            |
| **DNS Hostnames**      | Enabled                            |
| **Block Public Access**| Off                                |
| **IPv6 CIDR**          | —                                  |
| **Main Route Table**   | `rtb-04cb6119b6c81cb98`            |
| **Main Network ACL**   | `acl-03934f3284e2f1a0b`            |
| **DHCP Option Set**    | `dopt-0cca70265bc09d74e`           |
| **Owner ID**           | `660815084882`                     |

---

## Subnets

Both subnets reside in Availability Zone **`ap-northeast-1a`** within the Prashant-VPC.

### PublicSubnet

| Property                      | Value                                      |
|-------------------------------|--------------------------------------------|
| **Subnet ID**                 | `subnet-0433a130073b58c77`                 |
| **IPv4 CIDR**                 | `10.0.1.0/24`                              |
| **Available IPv4 Addresses**  | 250                                        |
| **Availability Zone**         | `ap-northeast-1a`                          |
| **Auto-assign Public IPv4**   | Yes                                        |
| **Route Table**               | `rtb-09401d36184bf9cf6` / PublicRouteTable |
| **State**                     | Available                                  |
| **Block Public Access**       | Off                                        |
| **Flow Logs**                 | None configured                            |

### PrivateSubnet

| Property                      | Value                                        |
|-------------------------------|----------------------------------------------|
| **Subnet ID**                 | `subnet-027665e65367e9aab`                   |
| **IPv4 CIDR**                 | `10.0.2.0/24`                                |
| **Available IPv4 Addresses**  | 251                                          |
| **Availability Zone**         | `ap-northeast-1a`                            |
| **Auto-assign Public IPv4**   | No                                           |
| **Route Table**               | `rtb-0ea47047101150188` / PrivateRouteTable  |
| **State**                     | Available                                    |
| **Block Public Access**       | Off                                          |
| **Flow Logs**                 | None configured                              |

---

## Route Tables

| Name               | ID                       | Associated Subnet  |
|--------------------|--------------------------|--------------------|
| PublicRouteTable   | `rtb-09401d36184bf9cf6`  | PublicSubnet       |
| PrivateRouteTable  | `rtb-0ea47047101150188`  | PrivateSubnet      |
| *(Main)*           | `rtb-04cb6119b6c81cb98`  | VPC default        |

---

## Network Connections

### Internet Gateway — Prashant-IGW

Attached to the VPC to allow inbound and outbound internet traffic for resources in the **PublicSubnet**.

### NAT Gateway — MyNATGateway

| Property                       | Value                                   |
|--------------------------------|-----------------------------------------|
| **NAT Gateway ID**             | `nat-04bd625a165bed04f`                 |
| **ARN**                        | `arn:aws:ec2:ap-northeast-1:660815084882:natgateway/nat-04bd625a165bed04f` |
| **Connectivity Type**          | Public                                  |
| **State**                      | Available                               |
| **Primary Public IPv4**        | `52.193.10.68`                          |
| **Primary Private IPv4**       | `10.0.1.217`                            |
| **Subnet**                     | `subnet-0433a130073b58c77` / PublicSubnet |
| **VPC**                        | `vpc-014b288abcb6e1b05` / Prashant-VPC  |
| **Primary Network Interface**  | `eni-0e6d045083c3cf5e5`                 |
| **Created**                    | Thursday, April 30, 2026 at 11:52:52 GMT+5:30 |
| **Secondary IPv4 Addresses**   | None                                    |

The NAT Gateway is placed in the **PublicSubnet** and enables outbound internet access for resources in the **PrivateSubnet** without exposing them to inbound traffic.

---

## Architecture Diagram

```
                        Internet
                           │
                    [Prashant-IGW]
                           │
              ┌────────────▼────────────┐
              │   PublicSubnet          │
              │   10.0.1.0/24           │
              │   (ap-northeast-1a)     │
              │                         │
              │   [MyNATGateway]        │
              │   Public IP: 52.193.10.68│
              └────────────┬────────────┘
                           │ (outbound only)
              ┌────────────▼────────────┐
              │   PrivateSubnet         │
              │   10.0.2.0/24           │
              │   (ap-northeast-1a)     │
              └─────────────────────────┘

              VPC: Prashant-VPC (10.0.0.0/16)
```

---

## Notes & Recommendations

- **Flow Logs** are not enabled on either subnet. Consider enabling them and shipping to CloudWatch or S3 for network traffic visibility and audit trails.
- **Single AZ deployment** — both subnets are in `ap-northeast-1a`. For high availability, consider adding a second AZ with additional public/private subnet pairs.
- **NAT Gateway costs** — the NAT Gateway incurs hourly and data-processing charges. If the private subnet workloads are dev/test only, consider shutting it down when not in use.
- The **Block Public Access** setting is Off for both subnets — ensure Security Groups and NACLs are correctly scoped to limit exposure.
- No **VPC Flow Logs** are configured at the VPC level either — recommended for production environments.

---

## Related Resources

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [NAT Gateway Pricing](https://aws.amazon.com/vpc/pricing/)
- [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
- [VPC Subnets Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)