# ☁️ AWS Detailed - Complete Guide

> A modular, beginner-to-advanced AWS reference guide covering all major services with theory, hands-on steps, architecture diagrams, interview questions, and best practices.

---

## 📁 Repository Structure

```
aws-detailed/
├── README.md                        ← You are here
├── services/
│   ├── 01_EC2.md                    ← Elastic Compute Cloud
│   ├── 02_S3.md                     ← Simple Storage Service
│   ├── 03_VPC.md                    ← Virtual Private Cloud + Subnets, IGW, NAT, Route Tables
│   ├── 04_IAM.md                    ← Identity & Access Management
│   ├── 05_EFS.md                    ← Elastic File System
│   ├── 06_EBS_Volumes.md            ← Elastic Block Store + Snapshots
│   ├── 07_ALB.md                    ← Application Load Balancer
│   ├── 08_Auto_Scaling.md           ← Auto Scaling Groups
│   ├── 09_CloudWatch.md             ← CloudWatch + Alarms + Metrics + Logs
│   ├── 10_Route53.md                ← DNS + Traffic Policies
│   ├── 11_CloudFront.md             ← CDN + Distribution
│   ├── 12_AMI.md                    ← Amazon Machine Images
│   ├── 13_Peering_Connection.md     ← VPC Peering
│   ├── 14_ENI_NIC.md                ← Elastic Network Interface
│   └── 15_Mini_Project_EC2.md       ← EC2 + ALB Mini Project
├── interview/
│   └── AWS_Interview_Guide.md       ← 200+ Interview Questions (All Levels)
└── diagrams/
    └── AWS_Architecture_Overview.md ← ASCII Architecture Diagrams
```

---

## 🗂️ Services Covered

| # | Service | Category | File |
|---|---------|----------|------|
| 1 | EC2 | Compute | [01_EC2.md](services/01_EC2.md) |
| 2 | S3 | Storage | [02_S3.md](services/02_S3.md) |
| 3 | VPC | Networking | [03_VPC.md](services/03_VPC.md) |
| 4 | IAM | Security | [04_IAM.md](services/04_IAM.md) |
| 5 | EFS | Storage | [05_EFS.md](services/05_EFS.md) |
| 6 | EBS + Snapshots | Storage | [06_EBS_Volumes.md](services/06_EBS_Volumes.md) |
| 7 | ALB | Networking | [07_ALB.md](services/07_ALB.md) |
| 8 | Auto Scaling | Compute | [08_Auto_Scaling.md](services/08_Auto_Scaling.md) |
| 9 | CloudWatch | Monitoring | [09_CloudWatch.md](services/09_CloudWatch.md) |
| 10 | Route 53 | DNS | [10_Route53.md](services/10_Route53.md) |
| 11 | CloudFront | CDN | [11_CloudFront.md](services/11_CloudFront.md) |
| 12 | AMI | Compute | [12_AMI.md](services/12_AMI.md) |
| 13 | VPC Peering | Networking | [13_Peering_Connection.md](services/13_Peering_Connection.md) |
| 14 | ENI / NIC | Networking | [14_ENI_NIC.md](services/14_ENI_NIC.md) |
| 15 | Mini Project | Project | [15_Mini_Project_EC2.md](services/15_Mini_Project_EC2.md) |

---

## 🎯 How to Use This Repo

- **Beginners** → Start with EC2 → S3 → IAM → VPC
- **Intermediate** → EFS, EBS, ALB, Auto Scaling, CloudWatch
- **Advanced** → Route53, CloudFront, VPC Peering, Architecture
- **Interview Prep** → [AWS Interview Guide](interview/AWS_Interview_Guide.md)

---

## 📌 Key AWS Concepts at a Glance

```
AWS Global Infrastructure
├── Regions (e.g., us-east-1, ap-south-1)
│   └── Availability Zones (AZ) (e.g., ap-south-1a, ap-south-1b)
│       └── Data Centers
├── Edge Locations (CloudFront CDN)
└── Local Zones
```

---

## 🔗 Quick Links

- [AWS Official Docs](https://docs.aws.amazon.com/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS Pricing Calculator](https://calculator.aws/)

---

> **Note:** Images/screenshots from lab sessions are in the `/images/` folder (add your own screenshots from AWS Console practice).

---

*Last Updated: May 2026 | Maintained for learning purposes*
