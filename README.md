# ☁️ AWS Detailed - Complete Guide

> A modular, beginner-to-advanced AWS reference guide covering all major services with theory, hands-on steps, architecture diagrams, interview questions, and best practices.

---

## ❓ What is Cloud Computing?

Cloud Computing is the **delivery of computing services** — servers, storage, databases, networking, software, and analytics — over the internet ("the cloud") to offer faster innovation, flexible resources, and economies of scale.

> Instead of owning and maintaining physical data centers and servers, you **rent** compute power, storage, and other services from a cloud provider and **pay only for what you use**.

### ☁️ Before Cloud vs After Cloud

| Before Cloud (On-Premise) | After Cloud (AWS) |
|---------------------------|-------------------|
| Buy physical servers | Rent virtual servers |
| Weeks to provision | Minutes to launch |
| Pay upfront (CapEx) | Pay as you go (OpEx) |
| Fixed capacity | Scale up/down instantly |
| You manage hardware | AWS manages hardware |
| Single location | Global in minutes |
| High maintenance cost | No maintenance overhead |

### 🏗️ Types of Cloud

| Type | Description | Example |
|------|-------------|---------|
| **Public Cloud** | Resources shared, owned by provider | AWS, Azure, GCP |
| **Private Cloud** | Dedicated to one organization | On-premise VMware |
| **Hybrid Cloud** | Mix of public + private | DB on-prem, app on AWS |
| **Multi-Cloud** | Using multiple cloud providers | AWS + GCP together |

### 📦 Cloud Service Models

```
IaaS (Infrastructure as a Service)
  → You get: VMs, storage, networking
  → You manage: OS, runtime, apps, data
  → AWS Example: EC2, EBS, VPC
  → Like: Renting an empty office space

PaaS (Platform as a Service)
  → You get: Runtime, OS, infrastructure
  → You manage: Only your application and data
  → AWS Example: Elastic Beanstalk, RDS, Lambda
  → Like: Renting a furnished office

SaaS (Software as a Service)
  → You get: Ready-to-use software
  → You manage: Nothing (just use it)
  → AWS Example: WorkMail, Chime, Gmail
  → Like: Using Gmail — just log in and use
```

### ⚡ 6 Key Advantages of Cloud Computing

1. **Trade CapEx for OpEx** — No upfront hardware investment; pay only when you consume
2. **Massive economies of scale** — AWS buys at huge volume → lower cost passed to you
3. **Stop guessing capacity** — Scale instantly up/down with actual demand
4. **Increase speed and agility** — Deploy globally in minutes, not weeks
5. **Stop spending on data centers** — Focus on your business, not server maintenance
6. **Go global in minutes** — One-click deploy to any AWS region worldwide

---

## ❓ What is AWS?

**Amazon Web Services (AWS)** is the world's most comprehensive and broadly adopted **cloud platform**, offering over **200 fully featured services** from data centers globally. AWS is owned by Amazon and launched in **2006**.

> AWS lets individuals, companies, and governments rent computing resources on demand — from a single virtual machine to powering AI-driven global applications.

### 📊 AWS by Numbers

| Metric | Value |
|--------|-------|
| Market Share | ~31% (largest cloud provider) |
| Global Regions | 33 regions worldwide |
| Availability Zones | 105+ AZs |
| Edge Locations | 400+ (CloudFront CDN) |
| Services | 200+ |
| Notable Customers | Netflix, NASA, Airbnb, Samsung, Swiggy |
| Launched | March 2006 |

### 🌍 AWS Global Infrastructure

```
AWS Global Infrastructure
│
├── Regions (33 worldwide)
│   ├── A geographic area (e.g., ap-south-1 = Mumbai)
│   ├── Completely independent — isolated from other regions
│   └── Choose based on: latency, compliance, cost
│
├── Availability Zones / AZ (105+)
│   ├── One or more physical data centers in a region
│   ├── Connected via low-latency private fiber
│   ├── e.g., ap-south-1a, ap-south-1b, ap-south-1c
│   └── Deploy across AZs for High Availability (HA)
│
├── Edge Locations (400+)
│   ├── Used by CloudFront CDN for content delivery
│   └── Cache content close to end users globally
│
└── Local Zones
    ├── Extensions of AWS regions to specific metro cities
    └── For ultra-low latency apps (gaming, AR/VR)
```

### 🗂️ AWS Services by Category

| Category | Key Services |
|----------|-------------|
| **Compute** | EC2, Lambda, ECS, EKS, Fargate |
| **Storage** | S3, EBS, EFS, Glacier |
| **Database** | RDS, Aurora, DynamoDB, ElastiCache |
| **Networking** | VPC, Route 53, CloudFront, ALB, NLB |
| **Security** | IAM, KMS, Shield, WAF, Cognito |
| **Monitoring** | CloudWatch, CloudTrail, X-Ray |
| **AI / ML** | SageMaker, Rekognition, Comprehend, Bedrock |
| **DevOps** | CodePipeline, CodeBuild, CodeDeploy |
| **Messaging** | SQS, SNS, EventBridge |
| **Analytics** | Athena, Redshift, Glue, Kinesis |

### 💰 AWS Pricing Philosophy

```
Pay-as-you-go   → Pay only for what you use, no upfront cost
Save when you commit → Reserve 1–3 years → up to 72% savings
Pay less with volume → More usage = lower per-unit price
Free Tier       → Many services free for 12 months (some forever)
```

### 🆚 AWS vs Azure vs GCP

| Feature | AWS | Azure | GCP |
|---------|-----|-------|-----|
| Market Share | ~31% | ~25% | ~11% |
| Launched | 2006 | 2010 | 2011 |
| Strengths | Most mature, broadest services | Microsoft/enterprise integration | Data, AI/ML |
| Compute | EC2 | Virtual Machines | Compute Engine |
| Object Storage | S3 | Blob Storage | Cloud Storage |
| Kubernetes | EKS | AKS | GKE |
| Serverless | Lambda | Azure Functions | Cloud Functions |
| Best For | General cloud workloads | Windows/.NET shops | Big data & ML |

### 🔐 AWS Shared Responsibility Model

```
AWS Responsible For (Security OF the Cloud):
  ├── Physical hardware & data centers
  ├── Global network infrastructure
  ├── Hypervisor / virtualization layer
  └── Managed service software (RDS engine, Lambda runtime)

Customer Responsible For (Security IN the Cloud):
  ├── Operating system patching (EC2)
  ├── Application security & code
  ├── IAM users, roles, and policies
  ├── Data encryption (at rest & in transit)
  ├── Network configuration (Security Groups, NACLs)
  └── Firewall rules and access controls
```

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
| 16 | Lambda | Serverless | [16_Lambda.md](services/16_Lambda.md) |
| 17 | EKS | Containers | [17_EKS.md](services/17_EKS.md) |
| 18 | RDS | Database | [18_RDS.md](services/18_RDS.md) |
| 19 | Route Tables + IGW + NAT | Networking | [19_Route_Tables_IGW_NAT.md](services/19_Route_Tables_IGW_NAT.md) |

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
