# 🎯 AWS Interview Guide - Complete Question Bank

> Covers: Simple → Intermediate → Advanced | Theory + Practical + Scenario-Based

---

## 📌 Table of Contents

1. [General AWS Concepts](#1-general-aws-concepts)
2. [EC2 Questions](#2-ec2---elastic-compute-cloud)
3. [S3 Questions](#3-s3---simple-storage-service)
4. [VPC Questions](#4-vpc---virtual-private-cloud)
5. [IAM Questions](#5-iam---identity--access-management)
6. [EBS & Snapshots](#6-ebs--snapshots)
7. [EFS Questions](#7-efs---elastic-file-system)
8. [Load Balancer Questions](#8-load-balancers)
9. [Auto Scaling Questions](#9-auto-scaling)
10. [CloudWatch Questions](#10-cloudwatch)
11. [Route 53 Questions](#11-route-53)
12. [CloudFront Questions](#12-cloudfront)
13. [Scenario-Based Questions](#13-scenario-based-questions)
14. [Practical / Hands-On Questions](#14-practical--hands-on-questions)
15. [Advanced Architecture Questions](#15-advanced-architecture-questions)

---

## 1. General AWS Concepts

### Simple Level

**Q1. What is AWS?**
> AWS (Amazon Web Services) is a cloud computing platform by Amazon that provides on-demand IT resources (servers, storage, databases, networking, AI) over the internet with pay-as-you-go pricing.

**Q2. What is Cloud Computing?**
> Cloud computing delivers computing services over the internet, eliminating the need to buy and maintain physical hardware. Types: IaaS, PaaS, SaaS.

**Q3. What are the types of cloud?**
> - **Public Cloud**: Resources shared among multiple users (AWS, Azure, GCP)
> - **Private Cloud**: Dedicated resources for one organization
> - **Hybrid Cloud**: Mix of public and private cloud

**Q4. What is IaaS, PaaS, SaaS? Give AWS examples.**

| Type | Description | AWS Example |
|------|-------------|-------------|
| IaaS | Infrastructure (VMs, storage) | EC2, EBS, VPC |
| PaaS | Platform (run code, no server management) | Elastic Beanstalk, Lambda |
| SaaS | Software over internet | WorkMail, Chime |

**Q5. What are AWS Regions and Availability Zones?**
> - **Region**: Geographic area with multiple data centers (e.g., ap-south-1 = Mumbai)
> - **AZ (Availability Zone)**: Isolated data center within a region (ap-south-1a, 1b, 1c)
> - Purpose: High availability and disaster recovery

**Q6. What is the AWS Free Tier?**
> Free usage limits for new AWS accounts for 12 months. Includes: 750 hours/month t2.micro EC2, 5 GB S3 storage, 1 million Lambda requests/month, etc.

**Q7. What is an ARN?**
> Amazon Resource Name — unique identifier for AWS resources.
> Format: `arn:aws:service:region:account-id:resource`
> Example: `arn:aws:s3:::my-bucket`

**Q8. What is the AWS Shared Responsibility Model?**
> - **AWS responsible for**: Security OF the cloud (hardware, global infrastructure, hypervisor)
> - **Customer responsible for**: Security IN the cloud (OS patching, data encryption, IAM, network config)

---

## 2. EC2 - Elastic Compute Cloud

### Simple Level

**Q1. What is EC2?**
> EC2 (Elastic Compute Cloud) is a web service providing resizable virtual servers (instances) in AWS cloud. You can launch, stop, terminate instances on demand.

**Q2. What are EC2 instance types? Give examples.**
> - **t** — General purpose burstable (t2.micro, t3.small) — dev/test
> - **m** — Balanced compute/memory (m5.large) — production
> - **c** — Compute optimized (c5.xlarge) — CPU intensive
> - **r** — Memory optimized (r5.large) — databases
> - **p/g** — GPU (p3.2xlarge) — ML/graphics

**Q3. What is an AMI?**
> Amazon Machine Image — a template containing OS, application server, and applications used to launch EC2 instances.

**Q4. What is a Key Pair in EC2?**
> A Key Pair (public + private key) used for secure SSH access to EC2 instances. AWS stores public key on instance, you keep private key (.pem file).

**Q5. What is a Security Group?**
> A virtual firewall at the instance level controlling inbound and outbound traffic. Stateful — if inbound is allowed, return traffic is automatically allowed.

### Intermediate Level

**Q6. What are EC2 pricing models? When to use each?**

| Model | When to Use | Savings |
|-------|-------------|---------|
| On-Demand | Unpredictable, short-term | 0% |
| Reserved | Predictable, 1-3 years | Up to 72% |
| Spot | Fault-tolerant, batch jobs | Up to 90% |
| Dedicated Host | Compliance, licensing | Varies |

**Q7. What is the difference between Stop and Terminate?**
> - **Stop**: Instance shuts down, EBS data preserved, public IP released (private IP preserved), no instance charges
> - **Terminate**: Instance deleted permanently, EBS root volume deleted (by default), all data lost

**Q8. What is User Data in EC2?**
> A bash script that runs automatically at first instance launch (as root). Used for bootstrapping — installing packages, starting services, etc.

**Q9. What is Elastic IP? When should you use it?**
> A static public IPv4 address. Use when you need a fixed IP that doesn't change when instance stops/starts. Free when associated with running instance, charged when allocated but unassociated.

**Q10. What is the difference between Security Groups and NACLs?**

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Order | All rules checked | Numbered order |

### Advanced Level

**Q11. What is EC2 Placement Groups? Types?**
> Controls placement of instances on underlying hardware:
> - **Cluster**: Same rack, low latency (HPC) — risk: single rack failure
> - **Spread**: Different racks, HA (critical apps) — max 7/AZ
> - **Partition**: Groups of instances on separate partitions (big data: HDFS, Kafka)

**Q12. What is an EC2 Instance Profile?**
> A container for an IAM role that can be attached to EC2 instances, allowing the instance to call AWS APIs without hardcoding credentials.

**Q13. What happens to an EC2 instance in a Spot interruption?**
> AWS sends a 2-minute warning before terminating. Should design for graceful shutdown — save state, drain connections. Can request Spot Blocks (uninterrupted for 1-6 hours, now deprecated).

**Q14. How would you move an EC2 instance from one region to another?**
> 1. Create AMI from instance
> 2. Copy AMI to target region
> 3. Launch new instance from copied AMI in target region

---

## 3. S3 - Simple Storage Service

### Simple Level

**Q1. What is S3?**
> Object storage service with unlimited storage. Stores files as objects with key (name) and value (data). 99.999999999% (11 nines) durability.

**Q2. What is an S3 Bucket?**
> A container for objects in S3. Name must be globally unique across all AWS accounts. Region-specific.

**Q3. What are S3 Storage Classes?**

| Class | Retrieval | Cost |
|-------|-----------|------|
| Standard | Immediate | High |
| Standard-IA | Immediate | Medium |
| Glacier | Minutes-Hours | Low |
| Glacier Deep Archive | 12-48 hours | Lowest |

**Q4. What is the maximum object size in S3?**
> 5 TB per object. For uploads > 5 GB, must use Multipart Upload.

### Intermediate Level

**Q5. What is S3 Versioning?**
> Keeps multiple versions of the same object. Protects against accidental deletion/overwrite. Once enabled, can only be suspended (not disabled).

**Q6. What is the difference between S3 Bucket Policy and ACL?**
> - **Bucket Policy**: JSON-based, resource-based policy for bucket and object access. More powerful.
> - **ACL**: Legacy access control at object/bucket level. AWS recommends disabling ACLs for most cases.

**Q7. How do you host a static website on S3?**
> 1. Create bucket, upload HTML/CSS files
> 2. Disable block public access
> 3. Enable ACLs, make objects public
> 4. Enable Static Website Hosting in Properties
> 5. Access via S3 website endpoint URL

**Q8. What is S3 Cross-Region Replication (CRR)?**
> Automatically replicates objects from source bucket to destination bucket in different region. Requires versioning enabled on both buckets.

**Q9. What is an S3 Presigned URL?**
> A temporary URL granting time-limited access to private S3 objects without requiring AWS credentials. Useful for sharing private files temporarily.
> ```bash
> aws s3 presign s3://my-bucket/file.pdf --expires-in 3600
> ```

**Q10. What is S3 Transfer Acceleration?**
> Uses CloudFront edge locations to speed up uploads to S3 from distant clients. Useful for global uploads.

### Advanced Level

**Q11. What is S3 Lifecycle Policy?**
> Automatically transitions objects between storage classes or expires them:
> Standard → Standard-IA (30 days) → Glacier (90 days) → Delete (365 days)

**Q12. What is S3 Object Lock?**
> Prevents object deletion/modification for a specified period. Modes:
> - **Governance**: Users with special permission can override
> - **Compliance**: No one can override (regulatory compliance)

**Q13. Explain S3 consistency model.**
> S3 provides **strong read-after-write consistency** for:
> - PUT of new objects
> - DELETE operations
> - Overwrite PUT (replaces old version)
> All fully consistent since December 2020.

---

## 4. VPC - Virtual Private Cloud

### Simple Level

**Q1. What is VPC?**
> A logically isolated virtual network in AWS where you can launch resources. You define IP ranges, subnets, routing, and security.

**Q2. What is a Subnet?**
> A segment of a VPC's IP address range where you can place AWS resources. Can be public (internet-facing) or private (internal only).

**Q3. What is an Internet Gateway (IGW)?**
> A horizontally scaled, redundant component attached to VPC that allows communication between instances in VPC and the internet.

**Q4. What is the difference between Public and Private Subnet?**
> - **Public**: Has route to IGW, instances can have public IPs, internet-accessible
> - **Private**: No direct internet route, uses NAT Gateway for outbound internet

### Intermediate Level

**Q5. What is a NAT Gateway? Why is it needed?**
> Network Address Translation Gateway — allows instances in private subnets to initiate outbound internet connections (for updates, API calls) while preventing inbound internet connections. Must be placed in public subnet.

**Q6. What is VPC Peering?**
> Private connection between two VPCs allowing resources to communicate using private IPs. Non-transitive — A↔B and B↔C does not allow A↔C.

**Q7. What is the difference between Security Groups and Route Tables?**
> - **Security Groups**: Firewall — controls what traffic is allowed to/from instances
> - **Route Tables**: Routing — controls where traffic is directed (to IGW, NAT, peering, etc.)

**Q8. Can you have overlapping CIDRs in VPC Peering?**
> No. VPCs being peered must have non-overlapping CIDR blocks.

**Q9. What is VPC Flow Logs?**
> Captures information about IP traffic going to/from network interfaces in VPC. Useful for troubleshooting and security monitoring. Can be sent to CloudWatch Logs or S3.

**Q10. What is a Bastion Host?**
> A hardened EC2 instance in a public subnet used as a jump server to SSH into private subnet instances. Also called Jump Box.
> ```
> Internet → Bastion Host (public) → Private EC2 (private)
> ```

### Advanced Level

**Q11. What is AWS Transit Gateway?**
> A network hub connecting multiple VPCs and on-premises networks. Supports transitive routing unlike VPC Peering. Best for many VPCs (hub-and-spoke model).

**Q12. What is VPC Endpoint?**
> Allows private connection from VPC to AWS services (S3, DynamoDB) without internet, IGW, or NAT. Types:
> - **Interface Endpoint**: ENI with private IP (most services)
> - **Gateway Endpoint**: Route table target (S3 and DynamoDB only — free)

**Q13. How would you design a 3-tier VPC architecture?**
> ```
> Public Subnet: ALB (load balancer)
> Private Subnet 1: Web/App servers (EC2 Auto Scaling)
> Private Subnet 2: Database (RDS Multi-AZ)
> NAT Gateway in public subnet for private subnet internet access
> ```

---

## 5. IAM - Identity & Access Management

### Simple Level

**Q1. What is IAM?**
> Service for controlling access to AWS resources. Manages authentication (who you are) and authorization (what you can do). Global service, no region.

**Q2. What are the IAM components?**
> - **Users**: Individual identities
> - **Groups**: Collections of users
> - **Roles**: Temporary permissions for services/users
> - **Policies**: JSON documents defining permissions

**Q3. What is the difference between IAM User and IAM Role?**
> - **User**: Long-term credentials (password, access keys) for humans
> - **Role**: Temporary credentials assumed by services, applications, or users. No permanent password/key.

**Q4. What is the principle of least privilege?**
> Give users/services only the minimum permissions needed to perform their job. Never grant more access than required.

### Intermediate Level

**Q5. What are the types of IAM policies?**
> - **AWS Managed**: Pre-built by AWS (e.g., AmazonS3FullAccess)
> - **Customer Managed**: Created by you, reusable
> - **Inline**: Directly attached to one user/group/role, not reusable

**Q6. What is an IAM Role? When to use it?**
> A set of permissions you can assume temporarily. Use for:
> - EC2 accessing S3 (instead of hardcoding access keys)
> - Lambda accessing DynamoDB
> - Cross-account access
> - Federated users (SSO)

**Q7. How does IAM policy evaluation work?**
> 1. Default: Deny everything
> 2. Explicit Allow: Grant access
> 3. Explicit Deny: **Always wins over Allow**
> Evaluation order: Explicit Deny > Explicit Allow > Default Deny

**Q8. What is MFA (Multi-Factor Authentication)?**
> Requires second form of authentication beyond password. Adds extra security layer. Recommended for all console users, required for root account.

**Q9. What are Access Keys? When should you NOT use them?**
> Long-term credentials (Access Key ID + Secret Access Key) for programmatic access. Avoid:
> - Embedding in code/scripts
> - Using on EC2 instances (use IAM Roles instead)
> - Sharing access keys between team members

### Advanced Level

**Q10. What is AWS STS (Security Token Service)?**
> Provides temporary security credentials for users/services. Used by:
> - IAM Roles (assume-role)
> - Federated users
> - Web Identity Federation
> Temp credentials expire automatically.

**Q11. What is Resource-Based Policy vs Identity-Based Policy?**
> - **Identity-based**: Attached to IAM user/group/role (what can this identity do?)
> - **Resource-based**: Attached to resource like S3 bucket (who can access this resource?)
> Cross-account access needs both or resource-based policy.

**Q12. What is IAM Access Analyzer?**
> Identifies resources shared with external entities (outside your account/org). Generates findings for S3 buckets, IAM roles, KMS keys, etc. that have cross-account or public access.

---

## 6. EBS & Snapshots

**Q1. What is EBS?**
> Elastic Block Store — persistent block-level storage volumes for EC2. Like a hard drive attached to your instance. Data persists after instance stop.

**Q2. What are EBS Volume Types?**
> - **gp3/gp2**: General Purpose SSD — default, most workloads
> - **io1/io2**: Provisioned IOPS SSD — critical databases
> - **st1**: Throughput HDD — big data, sequential reads
> - **sc1**: Cold HDD — infrequently accessed

**Q3. What is EBS Snapshot?**
> Point-in-time backup of EBS volume stored in S3 (managed by AWS). Incremental — only changed blocks saved after first snapshot.

**Q4. Can you mount an EBS volume to multiple instances?**
> By default, No. But **io1/io2 Multi-Attach** allows attachment to multiple EC2 instances in the same AZ simultaneously.

**Q5. What happens to EBS when EC2 is terminated?**
> Root EBS volume: Deleted by default (Delete on Termination = true)
> Additional EBS volumes: Preserved by default (Delete on Termination = false)
> Can change this behavior.

**Q6. How to encrypt an existing unencrypted EBS volume?**
> Direct encryption of existing volume is not possible. Steps:
> 1. Create snapshot of unencrypted volume
> 2. Copy snapshot with encryption enabled
> 3. Create new volume from encrypted snapshot
> 4. Detach old, attach new volume

**Q7. EBS vs Instance Store — differences?**

| Feature | EBS | Instance Store |
|---------|-----|----------------|
| Persistence | Yes | No (lost on stop) |
| Attach/Detach | Yes | No |
| Backup | Snapshots | Manual |
| Performance | Good | Excellent |

---

## 7. EFS - Elastic File System

**Q1. What is EFS?**
> Fully managed elastic NFS file system for Linux. Multiple EC2 instances can access it simultaneously. Auto-scales from 0 to petabytes.

**Q2. What protocol does EFS use?**
> NFS (Network File System) protocol, port 2049.

**Q3. EFS vs EBS — key differences?**
> - EFS: Multiple EC2s simultaneously, NFS, multi-AZ, Linux only, more expensive
> - EBS: Single EC2 (mostly), block storage, single AZ, all platforms

**Q4. What is an EFS Mount Target?**
> Network endpoint in each AZ that EC2 instances connect to for EFS access. Each AZ gets its own mount target with its own IP address.

**Q5. How to make EFS mount persistent across reboots?**
> Add entry to `/etc/fstab`:
> ```
> fs-12345.efs.region.amazonaws.com:/ /mnt/efs nfs4 defaults,_netdev 0 0
> systemctl daemon-reload && mount -a
> ```

**Q6. What are EFS performance modes?**
> - **General Purpose**: Default, web servers, CMS
> - **Max I/O**: Big data, media processing, hundreds of EC2 clients

---

## 8. Load Balancers

**Q1. What is Elastic Load Balancing (ELB)?**
> Automatically distributes incoming traffic across multiple targets (EC2, containers, IPs) to ensure high availability and fault tolerance.

**Q2. Types of Load Balancers in AWS?**

| Type | Layer | Protocol | Use Case |
|------|-------|----------|----------|
| ALB | 7 (App) | HTTP, HTTPS, gRPC | Web apps, microservices |
| NLB | 4 (Transport) | TCP, UDP, TLS | Low latency, gaming, IoT |
| CLB | 4+7 | HTTP, TCP | Legacy (avoid) |
| GWLB | 3 (Network) | IP | Network appliances |

**Q3. What is an ALB Listener?**
> Configuration on ALB to check for connection requests using protocol and port. Each listener has rules to route traffic.

**Q4. What is a Target Group?**
> Group of targets (EC2 instances, IPs, Lambda) that receive traffic from load balancer. Has health check configuration.

**Q5. What is path-based routing in ALB?**
> Route traffic based on URL path:
> - `/api/*` → API servers target group
> - `/images/*` → image servers target group
> - Default → main web servers

**Q6. What is host-based routing in ALB?**
> Route based on HTTP Host header:
> - `api.myapp.com` → API target group
> - `www.myapp.com` → web target group

**Q7. What is ALB Sticky Sessions?**
> Routes requests from same client to same target for duration of session. Uses cookies. Useful for stateful applications.

**Q8. What is Connection Draining (Deregistration Delay)?**
> When instance is deregistered/unhealthy, ALB stops sending new requests but allows existing in-flight requests to complete. Default: 300 seconds.

**Q9. ALB vs NLB — when to use which?**
> - **ALB**: HTTP/HTTPS apps, content-based routing, microservices, WebSocket
> - **NLB**: Extreme performance (millions of RPS), static IP, non-HTTP (TCP/UDP), gaming, IoT

---

## 9. Auto Scaling

**Q1. What is Auto Scaling?**
> Automatically adjusts number of EC2 instances based on demand. Ensures right capacity to handle load while minimizing cost.

**Q2. What is a Launch Template?**
> Template specifying EC2 instance configuration: AMI, instance type, key pair, security groups, user data. Used by Auto Scaling Groups.

**Q3. What are the capacity settings in ASG?**
> - **Desired**: Current target number of instances
> - **Minimum**: Floor — never go below this
> - **Maximum**: Ceiling — never exceed this

**Q4. Types of Auto Scaling Policies?**

| Policy | Description |
|--------|-------------|
| Target Tracking | Maintain metric at target (e.g., CPU = 65%) |
| Step Scaling | Scale in steps based on alarm magnitude |
| Scheduled | Scale at specific times |
| Predictive | ML-based forecasting |

**Q5. What is Cooldown Period?**
> Time after a scaling activity during which no further scaling occurs. Prevents rapid add/remove cycles. Default: 300 seconds.

**Q6. What is Instance Warmup Period?**
> Time for new instance to initialize before being counted in CloudWatch metrics for scaling. Prevents premature scale-out.

**Q7. What termination policies are available?**
> Default, OldestInstance, NewestInstance, OldestLaunchTemplate, ClosestToNextInstanceHour

**Q8. What is Predictive Scaling?**
> Uses ML to analyze historical patterns and proactively scale before traffic increases. Launches instances 14 minutes before predicted load.

---

## 10. CloudWatch

**Q1. What is CloudWatch?**
> AWS monitoring and observability service collecting metrics, logs, and events from AWS resources and applications.

**Q2. What metrics does CloudWatch collect for EC2 by default?**
> CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed
> ⚠️ Memory/RAM is NOT included by default (requires CloudWatch Agent)

**Q3. What is the difference between Basic and Detailed Monitoring?**
> - **Basic**: Free, 5-minute granularity
> - **Detailed**: Paid, 1-minute granularity

**Q4. What is a CloudWatch Alarm?**
> Monitors a metric and performs action when threshold is breached. States: OK, ALARM, INSUFFICIENT_DATA

**Q5. What actions can a CloudWatch Alarm trigger?**
> - EC2 actions (stop, start, reboot, terminate, recover)
> - Auto Scaling actions
> - SNS notifications
> - Systems Manager OpsItem

**Q6. What is CloudWatch Agent?**
> Software installed on EC2 to send custom metrics (memory, disk usage) and logs to CloudWatch. Required for RAM monitoring.

**Q7. What is CloudWatch Logs Insights?**
> Interactive query service to analyze CloudWatch Logs using a purpose-built query language. Find patterns, errors, and statistics in logs.

**Q8. What is the difference between CloudWatch and CloudTrail?**
> - **CloudWatch**: Performance monitoring, metrics, logs, alarms
> - **CloudTrail**: Audit log of API calls (who did what, when, from where)

---

## 11. Route 53

**Q1. What is Route 53?**
> AWS DNS web service providing domain registration, DNS routing, and health checking.

**Q2. What DNS record types does Route 53 support?**
> A, AAAA, CNAME, ALIAS, MX, TXT, NS, SOA, SRV, CAA, PTR

**Q3. What is the difference between CNAME and ALIAS record?**
> - **CNAME**: Points domain to another domain. Cannot be used for root domain (apex).
> - **ALIAS**: AWS-specific, points to AWS resources (ALB, CloudFront, S3). Can be used for root domain. Free, faster.

**Q4. What are Route 53 Routing Policies?**
> Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multivalue, IP-based

**Q5. What is a Hosted Zone?**
> Container for DNS records for a domain. Two types:
> - **Public**: Resolves queries from internet
> - **Private**: Resolves queries within VPC only

**Q6. What is Route 53 Health Check?**
> Monitors health of endpoints (HTTP, HTTPS, TCP). Used with failover routing to automatically route to healthy endpoints.

---

## 12. CloudFront

**Q1. What is CloudFront?**
> AWS CDN (Content Delivery Network) that delivers content globally using 400+ edge locations for low latency.

**Q2. What is an Edge Location?**
> Data center where CloudFront caches content, closer to users than the origin. 400+ worldwide.

**Q3. What is Origin in CloudFront?**
> Source of original content — can be S3, EC2, ALB, or custom HTTP server.

**Q4. What is CloudFront Invalidation?**
> Forces CloudFront to remove cached content before TTL expires. Useful after content updates.
> Cost: First 1000 paths/month free.

**Q5. What is OAC (Origin Access Control)?**
> Restricts S3 bucket access to only CloudFront (keeps S3 private). Replaces older OAI (Origin Access Identity).

**Q6. What is CloudFront + WAF?**
> AWS WAF can be attached to CloudFront to protect against SQL injection, XSS, DDoS at the edge.

**Q7. What is the difference between CloudFront Functions and Lambda@Edge?**
> - **CF Functions**: Lightweight JS, runs at edge, max 1ms, viewer req/resp only
> - **Lambda@Edge**: Full Node.js/Python, regional edge cache, all 4 events, up to 30s

---

## 13. Scenario-Based Questions

**Q1. Your EC2 instance can access the internet but you can't SSH into it. What do you check?**
> 1. Security Group — Is port 22 (SSH) allowed from your IP?
> 2. Key pair — Using correct .pem file?
> 3. Instance state — Is it running?
> 4. Public IP — Does it have one?
> 5. NACL — Is port 22 allowed at subnet level?
> 6. Route Table — Is there route to IGW?

**Q2. Users are complaining your website is slow globally. How do you fix it?**
> 1. Add **CloudFront CDN** in front of your origin
> 2. Enable **caching** for static assets
> 3. Use **Route 53 Latency-based routing** to route to nearest region
> 4. Enable **compression** (gzip/brotli) on CloudFront
> 5. Cache headers on origin server

**Q3. Your application needs to handle 100x traffic spike during a sale. How?**
> 1. **Auto Scaling** with aggressive target tracking (CPU 50%)
> 2. **Scheduled scaling** — pre-scale before sale
> 3. **ALB** to distribute traffic
> 4. **Predictive scaling** for historical patterns
> 5. **ElastiCache** to reduce DB load
> 6. **CloudFront** to cache static content
> 7. **RDS read replicas** for read traffic

**Q4. An EC2 in private subnet needs to download software packages from internet. How?**
> Create a **NAT Gateway** in public subnet, add route in private subnet route table: `0.0.0.0/0 → NAT Gateway`. Private EC2 can now reach internet outbound only.

**Q5. You need to share a large file with an external user who doesn't have AWS account. How?**
> Generate an **S3 Presigned URL** with expiry time. Share the URL — user can download without AWS credentials.
> ```bash
> aws s3 presign s3://bucket/file.zip --expires-in 86400
> ```

**Q6. Your EC2 instance keeps running even after the application crashes. What do you do?**
> 1. Create **CloudWatch Alarm** on application metric
> 2. Configure alarm action: **EC2 Action → Reboot/Recover**
> 3. Or use **CloudWatch Agent + custom metric** to detect app health
> 4. Set up **Auto Scaling health check** to replace unhealthy instances

**Q7. How do you ensure zero downtime during deployment?**
> 1. **Blue-Green Deployment**: Two identical environments, switch DNS/ALB
> 2. **Rolling Deployment**: Replace instances one by one (Auto Scaling)
> 3. **Canary Deployment**: Route small % to new version, monitor, then full rollout
> 4. **ALB + ASG Instance Refresh**: Gradually replace instances

**Q8. Database in private subnet, web app in public subnet — how do they communicate?**
> - They're in same VPC but different subnets
> - Web app uses **private IP** of database (no internet needed — same VPC)
> - Security Group on DB allows port 3306 from web server's Security Group
> - Route Table within VPC handles local routing automatically

**Q9. EC2 instances in two different VPCs need to communicate privately. How?**
> Create **VPC Peering Connection**:
> 1. Create peering connection (requester = VPC-A)
> 2. Accept in VPC-B
> 3. Update route tables in BOTH VPCs
> 4. Update security groups to allow traffic from other VPC CIDR

**Q10. S3 bucket was accidentally made public. How do you secure it immediately?**
> 1. S3 → Block Public Access → Edit → Enable ALL → Save (immediate effect)
> 2. Review and remove public ACLs
> 3. Review bucket policy — remove public statements
> 4. Enable S3 Access Analyzer to check for public access
> 5. Enable AWS Config rule: s3-bucket-public-read-prohibited

---

## 14. Practical / Hands-On Questions

**Q1. How do you SSH into an EC2 instance?**
> ```bash
> chmod 400 my-key.pem
> ssh -i "my-key.pem" ubuntu@<public-ip>    # Ubuntu
> ssh -i "my-key.pem" ec2-user@<public-ip>  # Amazon Linux
> ```

**Q2. How do you install and start Nginx on EC2?**
> ```bash
> sudo apt update && sudo apt install nginx -y
> sudo systemctl start nginx
> sudo systemctl enable nginx
> sudo systemctl status nginx
> ```

**Q3. How do you mount an EBS volume on Linux?**
> ```bash
> lsblk                              # List block devices
> fdisk /dev/xvdf                    # Partition: n, p, Enter, +5G, w
> mkfs.ext4 /dev/xvdf1               # Format
> mount /dev/xvdf1 /mnt              # Mount
> df -h                              # Verify
> ```

**Q4. How do you mount EFS on EC2?**
> ```bash
> apt install nfs-common -y
> mount -t nfs4 fs-xxx.efs.region.amazonaws.com:/ /mnt/efs
> df -h
> ```

**Q5. How do you create a snapshot via AWS CLI?**
> ```bash
> aws ec2 create-snapshot \
>   --volume-id vol-12345678 \
>   --description "My backup"
> ```

**Q6. How do you stress test an EC2 for Auto Scaling?**
> ```bash
> sudo apt install stress -y
> stress --cpu 100 --io 4 --vm 2 --vm-bytes 128M --timeout 300s
> # Watch ASG scale out as CPU > threshold
> ```

**Q7. How do you check if an EC2 instance has internet access?**
> ```bash
> ping 8.8.8.8          # Google DNS
> curl -I google.com    # HTTP request
> wget -q --spider http://google.com && echo OK
> ```

**Q8. How do you make fstab mount persistent?**
> ```bash
> # Get UUID
> blkid /dev/xvdf1
>
> # Edit fstab
> nano /etc/fstab
>
> # Add line:
> UUID=xxx /mnt ext4 defaults 0 2
>
> # Test
> mount -a && df -h
> ```

---

## 15. Advanced Architecture Questions

**Q1. Design a highly available 3-tier web application on AWS.**
> ```
> Tier 1 (Web):
>   Route 53 → CloudFront → ALB
>   ALB → Auto Scaling Group (EC2 web servers)
>   Subnets: Public, Multi-AZ
>
> Tier 2 (App):
>   ALB (internal) → Auto Scaling Group (EC2 app servers)
>   Subnets: Private, Multi-AZ
>
> Tier 3 (Database):
>   RDS Aurora (Multi-AZ, read replicas)
>   ElastiCache (Redis) for caching
>   Subnets: Private, Multi-AZ (isolated)
>
> Supporting:
>   S3 (static assets) + CloudFront CDN
>   ACM (SSL certificates)
>   WAF (web application firewall)
>   CloudWatch (monitoring + alarms)
>   VPC Flow Logs + CloudTrail (auditing)
> ```

**Q2. How do you implement disaster recovery on AWS?**
> - **Backup & Restore**: RTO hours, RPO hours — cheapest. Backup data to S3/Glacier.
> - **Pilot Light**: Minimal core running in DR region. Scale up in disaster.
> - **Warm Standby**: Scaled-down version running. Quick scale up.
> - **Multi-Site Active-Active**: Full production in two regions, Route 53 weighted. Zero RTO/RPO, most expensive.

**Q3. How would you reduce AWS costs for a production workload?**
> 1. **Reserved Instances** for predictable workloads (72% savings)
> 2. **Spot Instances** for batch/non-critical work (90% savings)
> 3. **S3 Lifecycle Policies** to move old data to Glacier
> 4. **Right-sizing** — analyze and downsize over-provisioned instances
> 5. **Auto Scaling** to scale in during off-peak
> 6. **CloudFront** to reduce EC2/origin requests
> 7. **S3 Intelligent-Tiering** for unknown access patterns
> 8. **Delete unused resources** (EIPs, EBS volumes, snapshots)

**Q4. What is the difference between RTO and RPO?**
> - **RTO (Recovery Time Objective)**: How long to restore service after disaster
> - **RPO (Recovery Point Objective)**: How much data loss is acceptable (max time between backups)

**Q5. How do you ensure data security at rest and in transit on AWS?**
> - **At Rest**: KMS encryption for EBS, S3, RDS, EFS. S3 SSE-S3/SSE-KMS.
> - **In Transit**: TLS/SSL for all connections. HTTPS on ALB/CloudFront. VPN/Direct Connect for on-premise. TLS for EFS mounts.
> - **Additional**: IAM permissions, VPC private subnets, Security Groups, Shield/WAF.

---

## 💡 Quick-Fire Common Interview Questions

| Question | Answer |
|----------|--------|
| Max S3 object size? | 5 TB |
| EC2 default SG behavior? | All outbound allowed, all inbound denied |
| EBS AZ restriction? | Must be same AZ as EC2 |
| Can S3 bucket name be reused? | Only after deleted + globally unique |
| Free tier EC2? | t2.micro or t3.micro, 750 hrs/month |
| CloudFront edge locations? | 400+ globally |
| Route 53 SLA? | 100% availability |
| IAM is global? | Yes, no region selection |
| EFS supports Windows? | No, Linux only |
| ALB max listeners? | 50 per LB |
| Default VPC CIDR? | 172.31.0.0/16 |
| IGW per VPC? | 1 only |
| AMI region specific? | Yes, must copy to other regions |

---

*Tip: Practice all hands-on steps on AWS Free Tier. Understanding comes from doing!*

---

## 16. Lambda - Serverless

**Q1. What is Lambda?**
> Serverless compute service that runs code in response to events without managing servers. AWS handles scaling, patching, and availability.

**Q2. What is a Cold Start in Lambda?**
> The latency when Lambda initializes a new container for first/idle invocation: pulls image → starts runtime → inits code → runs handler. Can be 100ms–5s. Warm starts reuse existing container (~1-50ms).

**Q3. What are Lambda triggers?**
> Event sources that invoke Lambda: API Gateway, S3, DynamoDB Streams, SQS, SNS, CloudWatch Events, ALB, Kinesis, Cognito, IoT.

**Q4. What is Lambda Execution Role?**
> IAM Role attached to Lambda granting it permission to access other AWS services (S3, DynamoDB, etc.). Lambda needs this to call any AWS API.

**Q5. Lambda vs EC2 — when to use each?**
> Lambda: Short-lived (< 15min), event-driven, variable traffic, simple functions. EC2: Long-running, stateful apps, need OS control, persistent connections.

**Q6. What is Provisioned Concurrency?**
> Pre-warms Lambda instances to eliminate cold starts. AWS keeps N containers running always. Costs more but guarantees low latency.

**Q7. What is a Lambda Layer?**
> A ZIP archive with shared libraries/dependencies that can be used across multiple Lambda functions. Avoids duplicating code.

**Q8. What is Lambda@Edge?**
> Run Lambda functions at CloudFront edge locations globally. Used for request/response manipulation, auth, A/B testing at the edge.

**Q9. Lambda timeout and memory limits?**
> Max timeout: 15 minutes. Memory: 128MB–10GB. CPU scales proportionally with memory.

**Q10. How does Lambda scale?**
> Lambda scales horizontally — each concurrent request gets its own container instance. Default limit: 1,000 concurrent executions per region.

---

## 17. EKS - Kubernetes

**Q1. What is EKS?**
> Elastic Kubernetes Service — fully managed Kubernetes control plane. AWS manages API server, etcd, scheduler. You manage worker nodes.

**Q2. What is a Pod?**
> Smallest deployable unit in Kubernetes. Contains one or more containers that share network and storage. Each pod gets its own IP in EKS.

**Q3. What is a Deployment?**
> Kubernetes object managing a set of replica pods. Handles rolling updates, rollbacks, desired state maintenance.

**Q4. What is the difference between a Service ClusterIP, NodePort, and LoadBalancer?**
> ClusterIP: Internal only (pod-to-pod). NodePort: Exposed on node's port (30000-32767). LoadBalancer: Creates AWS ELB automatically, public access.

**Q5. EKS node types — Managed vs Fargate?**
> Managed Node Groups: EC2 instances, AWS handles node lifecycle. Fargate: Serverless — no EC2 to manage, AWS runs each pod on dedicated micro-VM.

**Q6. What is IRSA?**
> IAM Roles for Service Accounts — assigns IAM roles to Kubernetes service accounts for pod-level AWS permissions. More granular than node-level IAM roles.

**Q7. What is kubectl?**
> Command-line tool for interacting with Kubernetes clusters. Used to deploy apps, inspect resources, manage cluster.

**Q8. What is HPA?**
> Horizontal Pod Autoscaler — automatically scales number of pod replicas based on CPU/memory metrics or custom metrics.

---

## 18. RDS - Relational Database

**Q1. What is RDS?**
> Fully managed relational database service supporting MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Aurora.

**Q2. What is Multi-AZ in RDS?**
> Synchronous replication to standby in another AZ. Automatic failover in 1-2 minutes if primary fails. Standby is NOT readable.

**Q3. What is a Read Replica?**
> Asynchronous read-only copy of primary DB for scaling reads. Can be promoted to standalone DB. Can be in same or different region.

**Q4. Multi-AZ vs Read Replica?**
> Multi-AZ: HA/failover, synchronous, not readable. Read Replica: read scaling, asynchronous, readable. Can have both simultaneously.

**Q5. How do automated backups work in RDS?**
> Daily full backup + 5-minute transaction log backup. Enables Point-in-Time Recovery to any second within retention period (1-35 days).

**Q6. What is RDS Proxy?**
> Managed database proxy that pools and shares connections. Critical for Lambda + RDS (Lambda can create thousands of connections overwhelming DB).

**Q7. What is Aurora?**
> AWS-native cloud DB engine: 5x faster than MySQL, 3x faster than Postgres. Auto-scales storage to 128TB. Up to 15 read replicas. 30-second failover.

**Q8. Can you SSH into RDS?**
> No. RDS is fully managed — AWS controls the underlying OS. You can only connect via database client protocols (MySQL, Postgres, etc.).

**Q9. How to encrypt existing unencrypted RDS?**
> Can't directly. Steps: Create snapshot → Copy snapshot with encryption → Restore from encrypted snapshot → Update app endpoint.

**Q10. What is RDS Performance Insights?**
> Database performance monitoring tool showing which SQL queries are consuming the most resources. Identifies slow queries and bottlenecks.

---

## 19. Route Tables, IGW, NAT Gateway

**Q1. What is an Internet Gateway?**
> A VPC component allowing bidirectional communication between VPC resources and the internet. One per VPC, fully redundant, free.

**Q2. What is a Route Table?**
> Set of rules determining where VPC network traffic is directed. Each subnet must be associated with exactly one route table.

**Q3. What is a NAT Gateway?**
> Allows private subnet instances to initiate outbound internet connections while blocking inbound internet-initiated connections. Must be in PUBLIC subnet.

**Q4. A private EC2 instance can't connect to internet. What do you check?**
> 1. NAT Gateway exists and is in PUBLIC subnet
> 2. Private subnet's route table has: 0.0.0.0/0 → nat-xxx
> 3. Public subnet's route table has: 0.0.0.0/0 → igw-xxx
> 4. NAT Gateway status = Available
> 5. Security Group allows outbound traffic

**Q5. What is the local route in a route table?**
> Automatically added route for VPC CIDR (e.g., 10.0.0.0/16 → local). Enables all subnets within same VPC to communicate. Cannot be removed.

**Q6. How many IGWs can you attach to a VPC?**
> Only ONE Internet Gateway per VPC. Attempting to attach a second gives an error.

**Q7. NAT Gateway vs NAT Instance?**
> NAT Gateway: AWS managed, highly available, up to 100Gbps, no SG, ~$32/month. NAT Instance: EC2 you manage, single AZ, limited bandwidth, has SG, cheaper.

**Q8. Why must NAT Gateway be in a PUBLIC subnet?**
> NAT Gateway itself needs internet access to forward private subnet traffic. Public subnet's route table has 0.0.0.0/0 → IGW, enabling NAT to reach internet.

**Q9. What is longest prefix match in routing?**
> When multiple routes match a destination, the most specific (longest) route wins. 10.0.1.0/24 beats 10.0.0.0/16 for packet to 10.0.1.5.

**Q10. Scenario: Lambda in VPC needs to access S3 and internet. What do you set up?**
> For S3: VPC Gateway Endpoint (free, no NAT needed). For internet: NAT Gateway in public subnet + route in Lambda's private subnet. Lambda needs private subnet with NAT route.
