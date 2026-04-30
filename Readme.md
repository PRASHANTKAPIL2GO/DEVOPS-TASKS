# AWS Infrastructure Examples and Tutorials

This repository contains a collection of AWS architecture examples, infrastructure setups, and deployment guides. Each folder contains a focused tutorial or sample configuration for a specific AWS service, design pattern, or operational task.

## Contents

- `AWS EC2 Auto Scaling Setup/`
  - Example configuration and documentation for using EC2 Auto Scaling groups.
  - Includes scaling policies, launch template guidance, and high availability considerations.

- `AWS S3 Bucket — Public Policy + Restricted Access Setup/`
  - Demonstrates how to configure S3 bucket policies for selective public access and restricted access.
  - Useful for hosting static content while protecting sensitive assets.

- `AWS S3 Bucket Policy — Upload Restrictions + Encryption Enforcement/`
  - Shows how to enforce upload restrictions and require server-side encryption on S3 buckets.
  - Ideal for compliance and secure file upload workflows.

- `AWS VPC Infrastructure with Terraform/`
  - Contains a VPC architecture deployment using Terraform.
  - Covers subnets, routing, security groups, gateways, and infrastructure-as-code best practices.

- `AWSWAFImplementationWithApplicationLoadBalancer/`
  - Demonstrates integration of AWS WAF with an Application Load Balancer.
  - Includes rules and policies for protecting web applications from common threats.

- `Bastion-less Architecture using AWS Systems Manager (SSM)/`
  - Shows how to manage EC2 instances without a bastion host using AWS SSM.
  - Focuses on secure remote management and session manager access.

- `Build a Hub-and-Spoke VPC Architecture/`
  - Presents a hub-and-spoke network topology for multi-account or multi-region AWS setups.
  - Includes central networking, shared services, and spoke VPC connectivity.

- `Centralized Logging Architecture on AWS/`
  - Contains examples for collecting and centralizing logs in AWS.
  - Likely includes CloudWatch Logs, S3 storage, or other logging aggregation methods.

- `CloudWatch Advanced Monitoring/`
  - Provides advanced monitoring techniques using Amazon CloudWatch.
  - Includes dashboards, alarms, and custom metrics for operational visibility.

- `Deploy EC2 Instance Using Terraform/`
  - Demonstrates provisioning EC2 instances with Terraform.
  - Covers instance configuration, networking, and infrastructure-as-code deployment.

- `Disaster Recovery Setup/`
  - Contains disaster recovery planning and architecture examples.
  - May include backup, failover, and recovery strategies for AWS resources.

- `EBS Snapshot, Volume Recovery & Data Persistence Verification/`
  - Shows how to use EBS snapshots and restore volumes.
  - Includes verification of data persistence and recovery procedures.

- `EC2 Instance Bootstrap with User Data Scripts/`
  - Covers how to bootstrap EC2 instances using user data.
  - Includes sample scripts for initial configuration and application deployment.

- `EC2 Instance with Nginx Web Server Setup/`
  - Demonstrates deploying an EC2 instance running Nginx.
  - Useful for simple web server setups and application hosting.

- `Enable CloudWatch Monitoring/`
  - Shows steps to enable CloudWatch monitoring on AWS resources.
  - Focuses on metrics collection, logs, and basic observability.

- `Implement Failover Routing/`
  - Contains examples for routing failover and resiliency.
  - Likely uses Route 53 or multi-AZ/multi-region routing techniques.

- `Implement IAM Best Practices/`
  - Provides IAM policy and access management best practices.
  - Helps design secure identity and access control for AWS accounts.

- `Implement Private EC2 with S3 Access via VPC Endpoint/`
  - Shows how to deploy private EC2 instances that access S3 through a VPC endpoint.
  - Useful for secure, private network architectures without internet egress.

- `Implement RDS Read Replica/`
  - Demonstrates setting up RDS read replicas for database scaling and redundancy.
  - Includes replication configuration and failover considerations.

- `RDS Deployment/`
  - Covers deploying Amazon RDS instances.
  - Includes database provisioning, networking, and basic configuration.

## How to Use This Repository

1. Choose the folder that matches the AWS scenario you want to explore.
2. Review the folder contents and any associated `Readme.md` or documentation files.
3. Follow the provided steps, code samples, or Terraform templates inside each directory.

## Notes

- Folder names describe the main topic for each example.
- Some directories may contain additional `Readme.md` files with more detailed instructions.
- This top-level README is intended as an index and overview of the repository structure.

