# Multi-Region ALB Setup — Primary & Secondary Load Balancers

## Overview

This project demonstrates a **multi-region Application Load Balancer (ALB)** setup with a Primary server in **Asia Pacific (Mumbai)** (`ap-south-1`) and a Secondary server in **US East (Ohio)** (`us-east-2`). Both load balancers serve web traffic over HTTP and are independently accessible via their respective ALB DNS endpoints.

---

## Architecture

```
         Internet
            │
   ┌────────┴─────────┐
   │                  │
[Primary ALB]    [Secondary ALB]
 ap-south-1        us-east-2
   │                  │
[PRIMARY SERVER]  [SECONDARY SERVER]
```

---

## Load Balancers

### Primary ALB

| Property              | Value                                                                 |
|-----------------------|-----------------------------------------------------------------------|
| **DNS Name**          | `primary-alb-1519082712.ap-south-1.elb.amazonaws.com`                |
| **Region**            | Asia Pacific (Mumbai) — `ap-south-1`                                 |
| **Response**          | `PRIMARY SERVER`                                                      |
| **Protocol**          | HTTP (Not secure)                                                     |

### Secondary ALB — `secondary-alb-2`

| Property                  | Value                                                                   |
|---------------------------|-------------------------------------------------------------------------|
| **Name**                  | secondary-alb-2                                                         |
| **ARN**                   | `arn:aws:elasticloadbalancing:us-east-2:660815084882:loadbalancer/app/secondary-alb-2/70778400f67caa11` |
| **DNS Name**              | `secondary-alb-2-1980134651.us-east-2.elb.amazonaws.com`               |
| **Region**                | US East (Ohio) — `us-east-2`                                            |
| **Load Balancer Type**    | Application                                                             |
| **Scheme**                | Internet-facing                                                         |
| **IP Address Type**       | IPv4                                                                    |
| **Status**                | Active ✅                                                               |
| **VPC**                   | `vpc-066de49f893673e4d`                                                 |
| **Hosted Zone**           | `Z3AADJGX6KTTL2`                                                       |
| **Availability Zones**    | `us-east-2b` (`subnet-05bc46abadeb4d04a`), `us-east-2a` (`subnet-00b59c0250ddf27df`) |
| **Date Created**          | April 27, 2026, 16:47 (UTC+05:30)                                       |
| **Response**              | `SECONDARY SERVER`                                                      |

---

## Secondary ALB — Listener Configuration

| Protocol | Port | Default Action                          | Target Group Stickiness | Security Policy | mTLS |
|----------|------|-----------------------------------------|-------------------------|-----------------|------|
| HTTP     | 80   | Forward to `secondary-tg-2` (1 rule, 100%) | Off                  | Not applicable  | Not applicable |

---

## Target Group — `secondary-tg-2`

| Property             | Value                                                                                  |
|----------------------|----------------------------------------------------------------------------------------|
| **ARN**              | `arn:aws:elasticloadbalancing:us-east-2:660815084882:targetgroup/secondary-tg-2/c593ed874dffecd5` |
| **Target Type**      | Instance                                                                               |
| **Protocol : Port**  | HTTP : 80                                                                              |
| **Protocol Version** | HTTP1                                                                                  |
| **IP Address Type**  | IPv4                                                                                   |
| **VPC**              | `vpc-066de49f893673e4d`                                                                |
| **Load Balancer**    | `secondary-alb-2`                                                                      |
| **Total Targets**    | 1                                                                                      |
| **Healthy**          | 1 ✅                                                                                   |
| **Unhealthy**        | 0                                                                                      |
| **Anomaly Mitigation** | Not applicable                                                                       |

### Registered Target

| Instance ID           | Name           | Port | Zone                    | Health Status | Admin Override | Launch Date  |
|-----------------------|----------------|------|-------------------------|---------------|----------------|--------------|
| `i-07a54d4d57c4571d7` | Secondary-Ser… | 80   | us-east-2a (use2-az1)   | Healthy ✅    | No override    | April 27, 2026 |

---

## Endpoints

| Server           | URL                                                                    |
|------------------|------------------------------------------------------------------------|
| Primary Server   | `http://primary-alb-1519082712.ap-south-1.elb.amazonaws.com`          |
| Secondary Server | `http://secondary-alb-2-1980134651.us-east-2.elb.amazonaws.com`       |

> ⚠️ Both endpoints are served over **HTTP (port 80)** with no TLS. Browsers will flag these as "Not secure."

---

## Notes & Recommendations

- **HTTPS not configured** — both ALBs serve traffic over plain HTTP. Add an ACM certificate and an HTTPS listener (port 443) with HTTP → HTTPS redirect for production use.
- **No failover/routing policy shown** — if this is a DR or active-passive setup, consider using **AWS Route 53** with health-check-based failover routing to automatically shift traffic from the primary to the secondary ALB on failure.
- **Single target per target group** — `secondary-tg-2` has only 1 registered instance. Adding more instances increases availability and enables the ALB to distribute load.
- **Target group stickiness is Off** — appropriate for stateless workloads; enable if sessions need to be pinned to a specific instance.
- **Anomaly mitigation is "Not applicable"** — anomaly detection requires at least 3 healthy targets; register additional instances to enable it.
- **No WAF attached to secondary ALB** — if WAF protection (like that on the primary) is required, associate `webACL1` or a new Web ACL with `secondary-alb-2`.

---

## Related Resources

- [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [Route 53 Failover Routing](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-failover.html)
- [ACM for HTTPS on ALB](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html)
- [ALB Target Groups](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)