# 🖥️ EC2 - Elastic Compute Cloud

## 📌 What is EC2?

Amazon EC2 (Elastic Compute Cloud) is a web service that provides **resizable virtual servers (instances)** in the AWS cloud. It allows you to run applications without investing in physical hardware.

> Think of EC2 as **renting a computer in the cloud** — you choose the OS, CPU, RAM, storage, and networking.

---

## 🏗️ EC2 Architecture

```
AWS Region (ap-south-1)
└── Availability Zone (ap-south-1a)
    └── VPC
        └── Subnet (Public/Private)
            └── EC2 Instance
                ├── AMI (OS Image)
                ├── Instance Type (CPU/RAM)
                ├── EBS Volume (Storage)
                ├── Security Group (Firewall)
                ├── Key Pair (SSH Access)
                └── Elastic IP (Static Public IP)
```

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **AMI** | Amazon Machine Image — blueprint/template for the instance (OS + pre-installed software) |
| **Instance Type** | Defines CPU, RAM, network capacity (e.g., t2.micro, m5.large) |
| **Key Pair** | SSH key for secure login (`.pem` file) |
| **Security Group** | Virtual firewall — controls inbound/outbound traffic |
| **Elastic IP** | Static public IP address assigned to instance |
| **User Data** | Bash script that runs at instance launch (bootstrapping) |
| **Placement Group** | Controls how instances are placed on underlying hardware |

---

## 📦 Instance Type Families

| Family | Use Case | Examples |
|--------|----------|---------|
| **t** (General/Burstable) | Dev, test, small apps | t2.micro, t3.small |
| **m** (General Purpose) | Balanced compute/memory | m5.large, m6i.xlarge |
| **c** (Compute Optimized) | CPU-intensive tasks | c5.large, c6g.2xlarge |
| **r** (Memory Optimized) | In-memory databases | r5.large, r6g.xlarge |
| **p/g** (GPU) | ML, graphics | p3.2xlarge, g4dn.xlarge |
| **i** (Storage Optimized) | High I/O databases | i3.large |
| **x** (Extra Large Memory) | SAP HANA, big DBs | x1e.32xlarge |

---

## 💰 EC2 Pricing Models

| Model | Description | Use Case | Savings |
|-------|-------------|----------|---------|
| **On-Demand** | Pay per hour/second, no commitment | Dev/test, unpredictable workloads | 0% (baseline) |
| **Reserved** | 1 or 3 year commitment | Production stable workloads | Up to 72% |
| **Spot** | Bid on unused capacity | Batch jobs, fault-tolerant | Up to 90% |
| **Savings Plans** | Flexible commitment | Mixed workloads | Up to 66% |
| **Dedicated Host** | Physical server for you | Compliance, licensing | Varies |

---

## 🛡️ Security Group vs NACL

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance level | Subnet level |
| State | Stateful | Stateless |
| Rules | Allow only | Allow + Deny |
| Direction | In + Out separate | In + Out separate |
| Evaluation | All rules checked | Rules in order |

---

## 🚀 EC2 Instance Lifecycle

```
[Pending] → [Running] → [Stopping] → [Stopped] → [Terminated]
                ↓
           [Rebooting]
```

| State | Description | Billing |
|-------|-------------|---------|
| Pending | Booting up | No charge |
| Running | Active | Charged |
| Stopping | Shutting down | No charge |
| Stopped | Off (EBS still exists) | EBS charged |
| Terminated | Deleted permanently | No charge |

---

## 🛠️ Hands-On: Launch EC2 Instance

### Step 1: Launch from Console
```
EC2 Dashboard → Launch Instance
  → Name: my-server
  → AMI: Ubuntu 22.04 LTS
  → Instance Type: t2.micro (Free Tier)
  → Key Pair: Create new → Download .pem
  → Security Group: Allow SSH (22), HTTP (80)
  → Storage: 8 GB gp3
  → Launch
```

### Step 2: Connect via SSH
```bash
# Change permission of key
chmod 400 my-key.pem

# Connect
ssh -i "my-key.pem" ubuntu@<public-ip>

# Switch to root
sudo -i

# Update packages
apt update && apt upgrade -y
```

### Step 3: Install Nginx Web Server
```bash
apt install nginx -y
systemctl start nginx
systemctl enable nginx
systemctl status nginx

# Verify: Open browser → http://<public-ip>
```

---

## 📜 User Data (Bootstrap Script)

User data runs **once at first launch** as root:

```bash
#!/bin/bash
apt update -y
apt install nginx -y
echo "<h1>Hello from EC2!</h1>" > /var/www/html/index.html
systemctl start nginx
systemctl enable nginx
```

> To pass in Launch Template: EC2 → Advanced Details → User Data

---

## 🌐 Elastic IP

```
EC2 → Elastic IPs → Allocate → Associate to Instance
```

- Static public IP that **doesn't change** on stop/start
- **Free** when associated with running instance
- **Charged** when allocated but NOT associated

---

## 📸 Hands-On: Install Custom Website Template

```bash
sudo -i
apt update
apt install nginx -y

# Download free template
wget https://www.free-css.com/assets/files/free-css-templates/download/page296/oxer.zip
ls

# Rename and extract
mv oxer.zip template.zip
apt install unzip -y
unzip template.zip

# Copy to nginx web root
cp -r oxer/* /var/www/html/

# Verify in browser: http://<public-ip>
```

---

## ✅ Advantages of EC2

- **Scalable** — Scale up/down in minutes
- **Flexible** — Choose OS, RAM, CPU
- **Cost-effective** — Pay only for what you use
- **Secure** — VPC, Security Groups, IAM
- **Global** — Available in all AWS regions
- **Integrated** — Works with S3, RDS, VPC, CloudWatch

---

## ❌ Disadvantages of EC2

- Requires management (patching, updates)
- Complex pricing model
- Not fully serverless — you manage the server
- Cold start time (~1-2 minutes to launch)

---

## 🏆 Best Practices

- Always use **IAM Roles** instead of access keys on EC2
- Use **Security Groups** to restrict access (least privilege)
- Enable **CloudWatch** monitoring for CPU, memory
- Use **Auto Scaling** to handle traffic spikes
- Regularly take **AMI snapshots** for backup
- Use **Private Subnets** for backend servers
- Enable **termination protection** on production instances
- Use **t3/t4g** over t2 for better performance

---

## 🔄 EC2 vs Lambda vs ECS

| Feature | EC2 | Lambda | ECS |
|---------|-----|--------|-----|
| Type | Virtual Machine | Serverless | Container |
| Managed By | You | AWS | You + AWS |
| Scaling | Manual/Auto | Automatic | Automatic |
| OS Access | Yes | No | Limited |
| Cost | Hourly | Per invocation | Per task |
| Use Case | Full control apps | Short functions | Microservices |

---

## 📚 Related Services

- **EBS** — Block storage for EC2
- **AMI** — Image/snapshot of EC2
- **Auto Scaling** — Scale EC2 automatically
- **ELB/ALB** — Load balance across EC2 instances
- **CloudWatch** — Monitor EC2 metrics
