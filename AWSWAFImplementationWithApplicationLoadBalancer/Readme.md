# webACL1 — AWS WAF Web ACL & ALB Security Setup

## Overview

`webACL1` is an AWS WAF (Web Application Firewall) Web ACL configured in the **Asia Pacific (Mumbai)** region (`ap-south-1`), managed under the AWS account **PRASHANT (1160-3630-7538)** via the `prashant_iam` IAM role. It is associated with an Application Load Balancer (`ALB1`) to protect web traffic against common threats including SQL injection attacks.

---

## Web ACL Details

| Property         | Value                                                   |
|------------------|---------------------------------------------------------|
| **Name**         | webACL1                                                 |
| **ARN**          | `arn:aws:wafv2:ap-south-1:582866...` *(truncated)*      |
| **ID**           | `40ed8970-f5e7-470d-8e65-44...` *(truncated)*           |
| **Region**       | Asia Pacific (Mumbai) — `ap-south-1`                    |
| **Region Scope** | CloudFront (Global) and Regional                        |
| **Rules**        | 3 rules configured                                      |
| **Logging**      | Not enabled                                             |
| **Account**      | PRASHANT (1160-3630-7538)                               |

---

## Associated Resource

| Property          | Value                                    |
|-------------------|------------------------------------------|
| **Name**          | ALB1 (Application Load Balancer)         |
| **Address Type**  | IPv4 (v4)                                |
| **VPC ID**        | `vpc-0d9f2ba84d9c0117c`                  |
| **Availability Zones** | 3 Availability Zones                |
| **Security Group**| `sg-04169b337380249d...` *(truncated)*   |
| **DNS Name**      | `ALB1-715654199.ap-south-...` *(truncated)* |
| **Date Created**  | April 3, 2026, 23:15 (UTC+05:30)         |

### ALB1 Integration Status

| Service                                   | Status               |
|-------------------------------------------|----------------------|
| Amazon CloudFront + AWS WAF               | Not integrated       |
| AWS Global Accelerator                    | Error: Status undetermined |
| AWS Config                                | Not integrated       |
| **AWS Web Application Firewall (WAF)**    | **Integrated ✅**    |

---

## WAF Rules

`webACL1` has **3 rules** configured. One confirmed managed rule group is detailed below.

### Rule 1 — AWS-AWSManagedRulesSQLiRuleSet

| Property    | Value                              |
|-------------|------------------------------------|
| **Type**    | AWS Managed Rule Group             |
| **Version** | Default (Version_1.3)              |
| **Capacity**| 200 WCUs                           |
| **Purpose** | Blocks request patterns associated with SQL injection attacks against databases |

#### Rule Overrides (all set to **Block**)

| Rule Name                            | Action |
|--------------------------------------|--------|
| `SQLiExtendedPatterns_QUERYARGUMENTS` | Block  |
| `SQLi_QUERYARGUMENTS`                | Block  |
| `SQLi_BODY`                          | Block  |
| `SQLi_COOKIE`                        | Block  |
| `SQLi_URIPATH`                       | Block  |

> All individual SQLi rules are set to **Block** mode. The "Override rule group" option (which would switch the entire group to Count mode) is **not** enabled.

---

## DDoS Protection

| Property                         | Value                   |
|----------------------------------|-------------------------|
| **Protection against low reputation sources** | Active under DDoS |
| **Scope**                        | Applies to Application Load Balancers only |

Resource-level DDoS protection is enabled using AWS managed threat lists. This applies specifically to the associated ALB (`ALB1`).

> **Note:** A console error (`Error calling wafv2:ListResourcesForWebACL`) was observed when viewing managed resources — this may be a transient permissions or API issue worth investigating.

---

## Notes & Recommendations

- **Logging is not enabled** on `webACL1`. Enabling WAF logging to S3 or CloudWatch Logs is strongly recommended for audit trails, incident response, and traffic analysis.
- **AWS CloudFront + WAF** and **AWS Config** are not integrated with ALB1. Consider enabling AWS Config for compliance monitoring of WAF rule changes.
- **AWS Global Accelerator** shows an undetermined status — investigate whether the integration is needed or if it's a permissions issue.
- The `ListResourcesForWebACL` API error suggests the IAM role (`prashant_iam`) may be missing the `wafv2:ListResourcesForWebACL` permission — review and update the IAM policy if needed.
- Consider adding additional managed rule groups such as `AWSManagedRulesCommonRuleSet` (Core Rule Set) or `AWSManagedRulesKnownBadInputsRuleSet` for broader protection.
- Review whether **Count mode** on any rules would be appropriate for testing before enforcing Block in production.

---

## Related Resources

- [AWS WAF Documentation](https://docs.aws.amazon.com/waf/)
- [AWS Managed Rule Groups](https://docs.aws.amazon.com/waf/latest/developerguide/aws-managed-rule-groups.html)
- [Enabling WAF Logging](https://docs.aws.amazon.com/waf/latest/developerguide/logging.html)
- [WAF with ALB](https://docs.aws.amazon.com/waf/latest/developerguide/waf-chapter.html)