# PrivateVPC — Private EC2 with S3 Gateway Endpoint

## Overview

`PrivateVPC` is an AWS VPC in the **Asia Pacific (Tokyo)** region (`ap-northeast-1`) designed around a private networking pattern: a private EC2 instance (`PrivateEC2`) with **no public IP**, no NAT Gateway, and S3 access routed entirely through a **VPC Gateway Endpoint** (`S3GatewayEndpoint`). Access to the instance is managed via **AWS SSM Session Manager** using the `PrivateEC2S3Role` IAM role.

---

## Architecture

```
                        Internet
                            │
                          [IGW]
                            │
              ┌─────────────▼──────────────┐
              │   PublicSubnet             │
              │   (ap-northeast-1a)        │
              └────────────────────────────┘

              ┌────────────────────────────┐
              │   PrivateSubnet            │
              │   (ap-northeast-1a)        │
              │                            │
              │   [PrivateEC2]             │
              │   10.0.2.192 (private only)│
              │   IAM: PrivateEC2S3Role    │
              └────────────┬───────────────┘
                           │
                  [S3GatewayEndpoint]
                  com.amazonaws.ap-northeast-1.s3
                           │
                        [S3]  (no internet path)
```

> ⚠️ No NAT Gateway is configured — private subnet resources have no outbound internet access except to S3 via the Gateway Endpoint.

---

## VPC Details

| Property               | Value                              |
|------------------------|------------------------------------|
| **VPC Name**           | PrivateVPC                         |
| **VPC ID**             | `vpc-0963c0303203e3170`            |
| **Region**             | Asia Pacific (Tokyo) — `ap-northeast-1` |
| **IPv4 CIDR**          | `10.0.0.0/16`                      |
| **State**              | Available                          |
| **Default VPC**        | No                                 |
| **Tenancy**            | Default                            |
| **DNS Resolution**     | Enabled                            |
| **DNS Hostnames**      | Enabled                            |
| **Block Public Access**| Off                                |
| **IPv6 CIDR**          | —                                  |
| **Main Route Table**   | `rtb-0dd6b0ebefb2edaea`            |
| **Main Network ACL**   | `acl-02d026a5b2a9c9daa`            |
| **DHCP Option Set**    | `dopt-0cca70265bc09d74e`           |
| **Owner ID**           | `660815084882`                     |

---

## Subnets

Both subnets are in Availability Zone **`ap-northeast-1a`**.

| Name          | Subnet ID                    | IPv4 CIDR    | Auto-assign Public IP | Route Table  |
|---------------|------------------------------|--------------|-----------------------|--------------|
| PublicSubnet  | `subnet-01b872875d23ded70`   | (from RT)    | Yes                   | PublicRT     |
| PrivateSubnet | `subnet-05513f0d7129bd58c`   | `10.0.2.0/24`| No                    | PrivateRT    |

---

## Route Tables

### PublicRT — `rtb-0a99482044b33a422`

| Destination  | Target                      | Status | Notes                    |
|--------------|-----------------------------|--------|--------------------------|
| `0.0.0.0/0`  | `igw-08781bbf883d3c038`     | Active | Internet via IGW         |
| `10.0.0.0/16`| local                       | Active | VPC-local traffic        |

- **Subnet association:** PublicSubnet (`subnet-01b872875d23ded70`)

### PrivateRT — `rtb-0662247816db0c245`

| Destination    | Target                        | Status | Notes                          |
|----------------|-------------------------------|--------|--------------------------------|
| `pl-61a54008`  | `vpce-0a11404e802ec527e`      | Active | S3 traffic via Gateway Endpoint|
| `10.0.0.0/16`  | local                         | Active | VPC-local traffic              |

- **Subnet association:** PrivateSubnet (`subnet-05513f0d7129bd58c`)
- No `0.0.0.0/0` route — **no internet access** from the private subnet.

---

## NAT Gateway

**No NAT Gateway configured.** The private subnet intentionally has no outbound internet path. S3 access is handled exclusively via the VPC Gateway Endpoint.

---

## VPC Endpoint — S3GatewayEndpoint

| Property                  | Value                                          |
|---------------------------|------------------------------------------------|
| **Endpoint ID**           | `vpce-0a11404e802ec527e`                       |
| **Name**                  | S3GatewayEndpoint                              |
| **Endpoint Type**         | Gateway                                        |
| **Service Name**          | `com.amazonaws.ap-northeast-1.s3`              |
| **Service Region**        | `ap-northeast-1`                               |
| **Status**                | Available ✅                                   |
| **VPC**                   | `vpc-0963c0303203e3170` (PrivateVPC)           |
| **IP Address Type**       | IPv4                                           |
| **DNS Record IP Type**    | service-defined                                |
| **Private DNS Names**     | Disabled                                       |
| **Associated Route Table**| PrivateRT (`rtb-0662247816db0c245`)            |
| **Associated Subnet**     | PrivateSubnet (`subnet-05513f0d7129bd58c`)     |
| **Created**               | Wednesday, April 29, 2026 at 19:34:56 GMT+5:30 |

---

## EC2 Instance — PrivateEC2

| Property                  | Value                                                                  |
|---------------------------|------------------------------------------------------------------------|
| **Name**                  | PrivateEC2                                                             |
| **Instance ID**           | `i-0c87816f578868fe5`                                                  |
| **Instance ARN**          | `arn:aws:ec2:ap-northeast-1:660815084882:instance/i-0c87816f578868fe5` |
| **Instance Type**         | `t2.micro`                                                             |
| **AMI ID**                | `ami-0f18986364089c4ab`                                                |
| **AMI Name**              | `al2023-ami-2023.11.20260413.0-kernel-6.1-x86_64`                     |
| **Platform**              | Linux/UNIX                                                             |
| **State**                 | Running ✅                                                             |
| **Public IPv4**           | — *(none)*                                                             |
| **Private IPv4**          | `10.0.2.192`                                                           |
| **Public DNS**            | — *(none)*                                                             |
| **Private DNS**           | `ip-10-0-2-192.ap-northeast-1.compute.internal`                        |
| **VPC**                   | `vpc-0963c0303203e3170` (PrivateVPC)                                   |
| **Subnet**                | `subnet-05513f0d7129bd58c` (PrivateSubnet)                             |
| **IAM Role**              | `PrivateEC2S3Role`                                                     |
| **Security Group**        | `sg-05aff2d2ef7a4f723` (private-ec2-sg)                               |
| **IMDSv2**                | Required                                                               |
| **Monitoring**            | Disabled                                                               |
| **Termination Protection**| Disabled                                                               |
| **Managed**               | false                                                                  |
| **Auto Scaling Group**    | None                                                                   |
| **Elastic IP**            | None                                                                   |

---

## Instance Access — SSM Session Manager

The instance has no public IP and no inbound SSH port open. Access is via **AWS SSM Session Manager** using the attached `PrivateEC2S3Role`.

| Property                         | Status           |
|----------------------------------|------------------|
| **SSM Ping Status**              | Offline ⚠️       |
| **Session Manager Connection**   | Not connected ⚠️ |
| **Last Ping Time**               | —                |
| **Agent Version**                | —                |

> **SSM is currently showing Offline / Not connected.** This is likely because the SSM agent on the instance cannot reach the SSM endpoint. Since there is no NAT Gateway and no internet access, the instance needs **SSM VPC Interface Endpoints** to use Session Manager. See troubleshooting below.

### Required VPC Endpoints for SSM (if not using NAT)

To use Session Manager without internet access, create Interface Endpoints for:

| Service                        | Endpoint Name                        |
|--------------------------------|--------------------------------------|
| SSM                            | `com.amazonaws.ap-northeast-1.ssm`   |
| SSM Messages                   | `com.amazonaws.ap-northeast-1.ssmmessages` |
| EC2 Messages                   | `com.amazonaws.ap-northeast-1.ec2messages` |

---

## Notes & Recommendations

- **SSM not working** — the most likely cause is missing SSM VPC Interface Endpoints. Add the three endpoints listed above to enable Session Manager in a fully private setup.
- **No NAT Gateway** — this is intentional for a cost-optimized private setup. S3 access works via the Gateway Endpoint. Any other AWS service access (SSM, CloudWatch, etc.) requires corresponding Interface Endpoints.
- **Monitoring disabled** — enable CloudWatch detailed monitoring for production workloads.
- **Termination protection is off** — consider enabling it to prevent accidental instance termination.
- **Flow logs not configured** — enable VPC Flow Logs for network visibility and audit compliance.
- **Single AZ** — both subnets are in `ap-northeast-1a`. Add a second AZ for high availability if needed.

---

## Related Resources

- [VPC Gateway Endpoints for S3](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)
- [SSM Session Manager without internet](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-prerequisites.html)
- [VPC Interface Endpoints for SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-create-vpc.html)
- [AWS Private EC2 Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html)