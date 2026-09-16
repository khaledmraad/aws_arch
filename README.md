# 🚀 Scalable Web Application with ALB and Auto Scaling

> A production-grade, highly available web application deployed on AWS using EC2 instances inside a properly architected VPC with public and private subnets across two Availability Zones.
---

##  Project Overview

This project demonstrates how to deploy a **production-grade, scalable, and highly available** web application on AWS. The architecture follows AWS Well-Architected Framework principles, leveraging:

- **High Availability** across multiple Availability Zones
- **Auto Scaling** to handle variable traffic loads
- **Security Best Practices** with private subnets, WAF, and least-privilege access
- **Performance Optimization** using CloudFront CDN caching
- **Managed Services** for database and monitoring

---

## 🏗️ Solution Architecture Diagram

![Scalable Web Application Architecture](./docs/architecture-diagram.png)

> **Figure 1:** End-to-end architecture showing traffic flow from Route 53 → CloudFront → ALB → EC2 (ASG) → RDS, with monitoring via CloudWatch/SNS and secure access via Systems Manager.

---

## 📐 Architecture Description

### Traffic Flow

1. **DNS Resolution** — Users access the application via a custom domain managed by **Route 53**, which resolves to the CloudFront distribution.
2. **CDN Caching** — **CloudFront** caches static assets (CSS, JS, images) at edge locations, reducing latency and origin load.
3. **Security Filtering** — **AWS WAF** inspects incoming requests and blocks OWASP Top 10 threats (SQL injection, XSS, etc.).
4. **Load Balancing** — The **Application Load Balancer (ALB)** distributes traffic across healthy EC2 instances in private subnets.
5. **Compute Layer** — **EC2 instances** managed by an **Auto Scaling Group (ASG)** dynamically scale based on CPU utilization and request count.
6. **Database Layer** — A **Multi-AZ RDS** instance (MySQL/PostgreSQL) provides synchronous replication and automated failover.
7. **Outbound Access** — EC2 instances reach the internet (for updates, APIs) via a **NAT Gateway** in the public subnet.
8. **Management** — **Systems Manager Session Manager** provides secure, bastion-free SSH access to instances.
9. **Monitoring** — **CloudWatch** collects metrics, triggers alarms, and publishes notifications via **SNS** (email/SMS).

---

## ️ Key AWS Services

| Service | Role |
|---------|------|
| **VPC** | Isolated network with public/private subnets, NAT Gateway, Security Groups, NACLs |
| **EC2 + ASG** | Compute layer with Launch Template, target tracking & step scaling policies |
| **ALB + WAF** | Layer 7 routing, SSL termination, WAF rules for OWASP Top 10 protection |
| **CloudFront** | Global CDN for caching static assets and reducing latency |
| **RDS Multi-AZ** | MySQL/PostgreSQL with automated failover and synchronous replication |
| **Route 53** | DNS management with alias records pointing to ALB/CloudFront, health checks |
| **Systems Manager** | Session Manager for secure, bastion-free instance access; Patch Manager |
| **CloudWatch + SNS** | Dashboards, custom metrics, alarms, and email/SMS notifications |
| **IAM** | Roles and policies following least-privilege principle |

---

##  Network Architecture

### VPC Design

VPC CIDR: 10.0.0.0/16
├── AZ-1
│ ├── Public Subnet: 10.0.1.0/24 (ALB, NAT Gateway)
│ ├── Private Subnet: 10.0.10.0/24 (EC2 Instances)
│ └── DB Subnet: 10.0.20.0/24 (RDS Primary)
└── AZ-2
├── Public Subnet: 10.0.2.0/24 (ALB, NAT Gateway)
├── Private Subnet: 10.0.11.0/24 (EC2 Instances)
└── DB Subnet: 10.0.21.0/24 (RDS Standby)



### Route Tables

| Subnet Type | Route Table | Destination | Target |
|-------------|-------------|-------------|--------|
| Public | `rtb-public` | 0.0.0.0/0 | Internet Gateway (IGW) |
| Private | `rtb-private` | 0.0.0.0/0 | NAT Gateway |
| DB | `rtb-db` | 10.0.0.0/16 | Local (VPC only — no internet) |

---

## 🔒 Security Architecture

### Defense-in-Depth Layers

1. **Network Layer**
   - Security Groups (stateful) — ALB allows only ports 80/443; EC2 allows only from ALB SG; RDS allows only from EC2 SG
   - NACLs (stateless) — Additional subnet-level filtering
   - Private subnets for EC2 and RDS (no direct internet access)

2. **Application Layer**
   - AWS WAF with AWS Managed Rules (OWASP Top 10, Known Bad Inputs, Amazon IP Reputation)
   - ALB listener rules with path-based and host-based routing
   - HTTPS enforced with ACM SSL/TLS certificates

3. **Data Layer**
   - RDS encryption at rest (KMS) and in transit (TLS)
   - Automated backups with 7-day retention
   - DB subnet group restricts access to application tier only

4. **Access Layer**
   - Systems Manager Session Manager (no SSH keys, no bastion host)
   - IAM roles with least-privilege policies
   - MFA enforced for console access




