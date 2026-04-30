# terraform-ec2 — Terraform-Provisioned EC2 Instance

## Overview

This project provisions an AWS EC2 instance (`Prashant-EC2`) in the **Asia Pacific (Tokyo)** region (`ap-northeast-1`) using Terraform. The instance was successfully created via `terraform apply` and is currently in a **Running** state.

---

## EC2 Instance Details

| Property                  | Value                                                              |
|---------------------------|--------------------------------------------------------------------|
| **Name**                  | Prashant-EC2                                                       |
| **Instance ID**           | `i-03196ec49a13ba16f`                                              |
| **Instance ARN**          | `arn:aws:ec2:ap-northeast-1:660815084882:instance/i-03196ec49a13ba16f` |
| **Instance Type**         | `t3.micro`                                                         |
| **AMI ID**                | `ami-0f18986364089c4ab`                                            |
| **Platform**              | Linux/UNIX                                                         |
| **Region**                | Asia Pacific (Tokyo) — `ap-northeast-1`                            |
| **State**                 | Running ✅                                                         |
| **Public IPv4 Address**   | `13.231.155.132`                                                   |
| **Private IPv4 Address**  | `172.31.41.172`                                                    |
| **Public DNS**            | `ec2-13-231-155-132.ap-northeast-1.compute.amazonaws.com`          |
| **Private DNS**           | `ip-172-31-41-172.ap-northeast-1.compute.internal`                 |
| **VPC ID**                | `vpc-0778a40b32fb6e11e`                                            |
| **Subnet ID**             | `subnet-045283f0a41c7586a`                                         |
| **IAM Role**              | None assigned                                                      |
| **IMDSv2**                | Required                                                           |
| **Monitoring**            | Disabled                                                           |
| **Managed**               | false                                                              |
| **Termination Protection**| (check console)                                                    |
| **Auto Scaling Group**    | None                                                               |
| **Elastic IP**            | None                                                               |
| **Account**               | MANAS (660815084882)                                               |

---

## Project Structure

```
terraform-ec2/
├── .terraform/               # Terraform plugin cache (auto-generated)
├── .terraform.lock.hcl       # Dependency lock file
├── main.tf                   # Main Terraform configuration
└── terraform.tfstate         # Local state file
```

---

## Terraform Configuration (`main.tf`)

```hcl
provider "aws" {
  region = "ap-northeast-1"
}

resource "aws_instance" "ec2" {
  ami           = "ami-0f18986364089c4ab"
  instance_type = "t3.micro"

  tags = {
    Name = "Prashant-EC2"
  }
}
```

---

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/install) installed
- AWS CLI configured with valid credentials:

```bash
aws configure
# AWS Access Key ID: <your-key>
# AWS Secret Access Key: <your-secret>
# Default region name: ap-northeast-1
# Default output format: json
```

---

## Usage

### 1. Initialize Terraform

```bash
terraform init
```

### 2. Preview Changes

```bash
terraform plan
```

### 3. Apply (Provision the Instance)

```bash
terraform apply
# Type 'yes' when prompted
```

Expected output:
```
Plan: 1 to add, 0 to change, 0 to destroy.
aws_instance.ec2: Creating...
aws_instance.ec2: Still creating... [00m10s elapsed]
aws_instance.ec2: Creation complete after 15s [id=i-03196ec49a13ba16f]
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

### 4. Destroy (Teardown)

```bash
terraform destroy
# Type 'yes' when prompted
```

---

## Notes & Recommendations

- **No IAM Role assigned** — if the instance needs to interact with other AWS services (S3, SSM, etc.), attach an appropriate IAM instance profile.
- **No Elastic IP** — the current public IP (`13.231.155.132`) is dynamic and will change on stop/start. Allocate and associate an Elastic IP if a stable public address is needed.
- **Monitoring is disabled** — enable detailed CloudWatch monitoring for production workloads.
- **IMDSv2 is required** — good security practice; ensure any application code uses IMDSv2-compatible metadata calls.
- **State is stored locally** (`terraform.tfstate`) — for team use, migrate state to a remote backend (e.g., S3 + DynamoDB for locking).
- **No Key Pair configured** — if SSH access is needed, add a `key_name` argument to the `aws_instance` resource and ensure the security group allows port 22.
- **No Security Group specified** — the instance is using the default VPC security group. Review inbound/outbound rules to ensure least-privilege access.

---

## Related Resources

- [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
- [AWS EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Terraform Remote State (S3 Backend)](https://developer.hashicorp.com/terraform/language/backend/s3)