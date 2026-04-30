# mydb-instance — RDS MySQL with AppServer EC2

## Overview

This setup provisions an **Amazon RDS MySQL** instance (`mydb-instance`) in **Asia Pacific (Tokyo)** (`ap-northeast-1`), accessed from a connected **EC2 AppServer** (`i-099b7f10723965a27`). The RDS instance is **not publicly accessible** and is reachable only from within the VPC. The MySQL client is installed on the AppServer to connect to the database from within the private network.

---

## Architecture

```
          Internet
              │
        [AppServer EC2]
        Public IP: 54.95.174.118
        Private IP: 10.0.1.107
        VPC: vpc-0d83170320fa4c325
              │
        (VPC-internal, port 3306)
              │
        [mydb-instance RDS]
        Endpoint: mydb-instance.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com
        Publicly Accessible: No
        VPC: vpc-0d83170320fa4c325
```

---

## RDS Instance — `mydb-instance`

### Summary

| Property           | Value                  |
|--------------------|------------------------|
| **DB Identifier**  | `mydb-instance`        |
| **Status**         | Available ✅           |
| **Role**           | Instance               |
| **Engine**         | MySQL Community        |
| **Class**          | `db.t3.micro`          |
| **CPU**            | ~4.18%                 |
| **Current Activity**| 0 Connections         |
| **Region & AZ**    | `ap-northeast-1c`      |

### Connection Details

| Property              | Value                                                                    |
|-----------------------|--------------------------------------------------------------------------|
| **Database Name**     | `mysql`                                                                  |
| **Master Username**   | `admin`                                                                  |
| **Port**              | `3306`                                                                   |
| **Endpoint**          | `mydb-instance.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com`           |
| **Endpoint Type**     | Instance endpoint                                                        |
| **Internet Access Gateway** | Disabled                                                           |
| **IAM Authentication**| Disabled                                                                 |

### Connectivity & Security

| Property                        | Value                                                        |
|---------------------------------|--------------------------------------------------------------|
| **Availability Zone**           | `ap-northeast-1c`                                            |
| **VPC**                         | `vpc-0d83170320fa4c325`                                      |
| **Subnet Group**                | `mysql-db-subnet`                                            |
| **Subnets**                     | `subnet-019bc5686c6664d30`, `subnet-06256a8a5a5a0fa9d`       |
| **Network Type**                | IPv4                                                         |
| **VPC Security Group**          | `terraform-202604291230284872000000002` (`sg-035d519c861598bbf`) — Active |
| **Publicly Accessible**         | **No** ✅                                                    |
| **Certificate Authority**       | `rds-ca-rsa2048-g1`                                          |
| **CA Date**                     | May 26, 2061, 04:24 (UTC+05:30)                              |
| **DB Instance Cert Expiry**     | April 29, 2027, 18:09 (UTC+05:30)                            |

---

## DB Subnet Group — `mysql-db-subnet`

| Property                | Value                                                          |
|-------------------------|----------------------------------------------------------------|
| **Name**                | `mysql-db-subnet`                                              |
| **ARN**                 | `arn:aws:rds:ap-northeast-1:660815084882:subgrp:mysql-db-subnet` |
| **VPC**                 | `vpc-0d83170320fa4c325`                                        |
| **Supported Networks**  | IPv4                                                           |
| **Description**         | Managed by Terraform                                           |
| **Tag**                 | `Name = DBSubnetGroup`                                         |

### Subnets

| Availability Zone | Subnet ID                      | CIDR Block    |
|-------------------|--------------------------------|---------------|
| `ap-northeast-1c` | `subnet-019bc5686c6664d30`     | `10.3.0.0/24` |
| `ap-northeast-1a` | `subnet-06256a8a5a5a0fa9d`     | `10.2.0.0/24` |

---

## EC2 AppServer — `i-099b7f10723965a27`

| Property                  | Value                                                                      |
|---------------------------|----------------------------------------------------------------------------|
| **Name**                  | AppServer                                                                  |
| **Instance ID**           | `i-099b7f10723965a27`                                                      |
| **Instance ARN**          | `arn:aws:ec2:ap-northeast-1:660815084882:instance/i-099b7f10723965a27`     |
| **Instance Type**         | `t2.micro`                                                                 |
| **AMI ID**                | `ami-0f18986364089c4ab`                                                    |
| **AMI Name**              | `al2023-ami-2023.11.20260413.0-kernel-6.1-x86_64`                         |
| **Platform**              | Linux/UNIX                                                                 |
| **State**                 | Running ✅                                                                 |
| **Public IPv4**           | `54.95.174.118`                                                            |
| **Private IPv4**          | `10.0.1.107`                                                               |
| **Public DNS**            | —                                                                          |
| **Private DNS**           | `ip-10-0-1-107.ap-northeast-1.compute.internal`                            |
| **VPC**                   | `vpc-0d83170320fa4c325`                                                    |
| **Subnet**                | `subnet-0d3ff4cd8273f56fd`                                                 |
| **IAM Role**              | None                                                                       |
| **IMDSv2**                | Required                                                                   |
| **Monitoring**            | Disabled                                                                   |
| **Termination Protection**| Disabled                                                                   |

---

## Security Group — AppServer (`sg-076764b0c2252065f`)

**Name:** `terraform-202604291230284872000000002`  
**Description:** Managed by Terraform  
**VPC:** `vpc-0d83170320fa4c325`

### Inbound Rules

| Protocol | Type | Port | Source      | Notes                        |
|----------|------|------|-------------|------------------------------|
| TCP      | HTTP | 80   | `0.0.0.0/0` | Open to internet ⚠️          |
| TCP      | SSH  | 22   | `0.0.0.0/0` | Open to internet ⚠️          |

### Outbound Rules

| Count | Notes               |
|-------|---------------------|
| 1     | Default allow-all   |

---

## MySQL Client Setup on AppServer

The MySQL client (`mysql-client-8.0`) was installed on the AppServer via AWS CloudShell / EC2 terminal:

```bash
sudo apt update
sudo apt install mysql-client -y
```

**Packages installed:**
- `mysql-client` (8.0.45)
- `mysql-client-8.0` (8.0.45)
- `mysql-client-core-8.0` (8.0.45)
- `mysql-common` (5.8+1.1.0build1)

---

## Connecting to the RDS Instance

### From AppServer (SSL)

```bash
# Step 1: Download SSL certificate bundle
curl -o global-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# Step 2: Connect with SSL verification
mysql -h mydb-instance.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com \
      -P 3306 \
      -u admin -p \
      --ssl-mode=VERIFY_IDENTITY \
      --ssl-ca=./global-bundle.pem
```

---

## Notes & Recommendations

- **SSH open to `0.0.0.0/0`** — port 22 is open to the entire internet on the AppServer security group. Restrict this to your specific IP or use AWS SSM Session Manager instead of SSH.
- **HTTP open to `0.0.0.0/0`** — port 80 is world-accessible. Scope this to known sources or put a load balancer in front.
- **No IAM role on AppServer** — if the EC2 needs AWS SDK/CLI access (e.g., for S3, SSM), attach an instance profile with appropriate permissions.
- **Monitoring disabled** — enable CloudWatch detailed monitoring on the EC2 and RDS for production observability.
- **Termination protection is off** — consider enabling it on both EC2 and RDS for production.
- **RDS certificate expires April 29, 2027** — plan rotation before that date to avoid connection failures.
- **DB subnet group spans two AZs** (`ap-northeast-1c` and `ap-northeast-1a`) — good multi-AZ readiness; consider enabling Multi-AZ on the RDS instance for automatic failover.
- **IAM Authentication is disabled** — for enhanced security, enable IAM DB Authentication to eliminate password-based access.

---

## Related Resources

- [RDS MySQL with SSL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/mysql-ssl-connections.html)
- [Amazon RDS Security Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.Security.html)
- [EC2 Security Group Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/security-group-rules.html)
- [RDS DB Subnet Groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)