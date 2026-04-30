# MySQL RDS — Primary DB with Read Replica

## Overview

This setup provisions an **Amazon RDS MySQL** primary instance (`mysql-primary-db`) with a **Read Replica** (`mysql-read-replica`) in the **Asia Pacific (Tokyo)** region (`ap-northeast-1`), managed under the AWS account **MANAS (660815084882)** via the `terraform-prashant` IAM role. Both instances are publicly accessible and share the same VPC, subnet group, and security group.

---

## Architecture

```
                    Application / IDE
                           │
              ┌────────────┴────────────┐
              │                         │
     [mysql-primary-db]        [mysql-read-replica]
       (Read + Write)             (Read Only)
       ap-northeast-1c            ap-northeast-1c
              │                         │
              └────────────┬────────────┘
                           │
                  VPC: vpc-03f3b407b2312ebf2
                  Subnet group: db-subnet-group
                  Security group: rds-sg
```

---

## Databases Summary

| DB Identifier        | Role    | Engine     | Status    | Size        | Region        | Upgrade Rollout |
|----------------------|---------|------------|-----------|-------------|---------------|-----------------|
| `mysql-primary-db`   | Primary | MySQL Community | Available ✅ | db.t3.micro | ap-northeast-1 | SECOND        |
| `mysql-read-replica` | Replica | MySQL Community | Available ✅ | db.t3.micro | ap-northeast-1 | SECOND        |

---

## Primary DB — `mysql-primary-db`

### Connection Details

| Property              | Value                                                          |
|-----------------------|----------------------------------------------------------------|
| **Database Name**     | `mysql`                                                        |
| **Master Username**   | `admin`                                                        |
| **Port**              | `3306`                                                         |
| **Endpoint**          | `mysql-primary-db.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com` |
| **Endpoint Type**     | Instance endpoint                                              |
| **Internet Access Gateway** | Disabled                                               |

### Connectivity & Security

| Property                        | Value                                    |
|---------------------------------|------------------------------------------|
| **Availability Zone**           | `ap-northeast-1c`                        |
| **VPC**                         | `vpc-03f3b407b2312ebf2`                  |
| **Subnet Group**                | `db-subnet-group`                        |
| **Subnets**                     | `subnet-06d8ba8682ed7452b`, `subnet-02c47166421b4f2b7` |
| **Network Type**                | IPv4                                     |
| **VPC Security Group**          | `rds-sg` (`sg-02d9e7b9446d1d5cf`) — Active |
| **Publicly Accessible**         | Yes                                      |
| **Certificate Authority**       | `rds-ca-rsa2048-g1`                      |
| **CA Date**                     | May 26, 2061, 04:24 (UTC+05:30)          |
| **DB Instance Cert Expiry**     | April 30, 2027, 01:20 (UTC+05:30)        |

---

## Read Replica — `mysql-read-replica`

### Connection Details

| Property              | Value                                                             |
|-----------------------|-------------------------------------------------------------------|
| **Database Name**     | `mysql`                                                           |
| **Master Username**   | `admin`                                                           |
| **Port**              | `3306`                                                            |
| **Endpoint**          | `mysql-read-replica.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com` |
| **Endpoint Type**     | Instance endpoint                                                 |
| **Internet Access Gateway** | Disabled                                                  |

### Connectivity & Security

| Property                        | Value                                    |
|---------------------------------|------------------------------------------|
| **Availability Zone**           | `ap-northeast-1c`                        |
| **VPC**                         | `vpc-03f3b407b2312ebf2`                  |
| **Subnet Group**                | `db-subnet-group`                        |
| **Subnets**                     | `subnet-06d8ba8682ed7452b`, `subnet-02c47166421b4f2b7` |
| **Network Type**                | IPv4                                     |
| **VPC Security Group**          | `rds-sg` (`sg-02d9e7b9446d1d5cf`) — Active |
| **Publicly Accessible**         | Yes                                      |
| **Certificate Authority**       | `rds-ca-rsa2048-g1`                      |
| **CA Date**                     | May 26, 2061, 04:24 (UTC+05:30)          |
| **DB Instance Cert Expiry**     | April 30, 2027, 01:30 (UTC+05:30)        |

---

## Read Replica — Monitoring Snapshot

Metrics observed for `mysql-read-replica` (approximate snapshot around 17:30–20:00):

| Metric                  | Value      | Notes                                      |
|-------------------------|------------|--------------------------------------------|
| **CPU Utilization**     | ~23%       | Moderate — monitor if it climbs further    |
| **Database Connections**| 1          | Single active connection                   |
| **Disk Queue Depth**    | 0.12       | Low — healthy I/O queue                   |
| **EBS Byte Balance %**  | 100%       | Full EBS burst balance available           |
| **EBS IO Balance %**    | 100%       | Full EBS IO burst balance available        |
| **Freeable Memory**     | 152.8 MB   | Low on a `db.t3.micro` — watch this closely |

---

## Connecting to the Databases

### MySQL CLI

```bash
# Primary (read/write)
mysql -h mysql-primary-db.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com \
      -u admin -p --port 3306

# Read Replica (read only)
mysql -h mysql-read-replica.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com \
      -u admin -p --port 3306
```

### Application Connection String (example)

```
# Primary
mysql://admin:<password>@mysql-primary-db.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com:3306/mysql

# Replica
mysql://admin:<password>@mysql-read-replica.cvcycegk2k2d.ap-northeast-1.rds.amazonaws.com:3306/mysql
```

---

## Notes & Recommendations

- **Publicly accessible is ON** — both instances are reachable from the internet. Ensure the `rds-sg` security group restricts inbound port 3306 to known IP ranges only. For production, set `Publicly Accessible` to No and access via a bastion or VPC-internal resources.
- **Freeable memory is low (152.8 MB)** — `db.t3.micro` has limited RAM. Monitor for memory pressure; consider upgrading to `db.t3.small` or above if workload grows.
- **Both instances are in the same AZ (`ap-northeast-1c`)** — this provides no AZ-level redundancy for the replica. Consider placing the replica in a different AZ for resilience.
- **DB certificate expires April 30, 2027** — plan rotation before expiry to avoid connection failures.
- **Automated backups** — verify backup retention is configured under the Maintenance & backups tab.
- **No Multi-AZ** — the primary is a single instance with no automatic failover. For production, enable Multi-AZ deployment on the primary.
- **CPU at 23% on replica** — healthy for now, but monitor if replication lag increases under heavier write loads on the primary.

---

## Related Resources

- [Amazon RDS MySQL Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
- [RDS Read Replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html)
- [RDS Multi-AZ Deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [RDS Security Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.Security.html)