# 📁 EFS - Elastic File System

## 📌 What is EFS?

Amazon EFS (Elastic File System) is a **fully managed, scalable, shared file storage** service that can be mounted by **multiple EC2 instances simultaneously** using the NFS protocol.

> Think of EFS as a **shared network drive (like Google Drive for EC2 instances)** — multiple servers can read/write to the same folder at the same time.

---

## 🏗️ EFS Architecture

```
AWS Region
└── EFS File System
    ├── Mount Target (AZ-1: ap-south-1a) → IP: 192.168.0.x
    ├── Mount Target (AZ-2: ap-south-1b) → IP: 192.168.1.x
    └── Mount Target (AZ-3: ap-south-1c) → IP: 192.168.2.x

EC2 Instance (AZ-1) ──┐
EC2 Instance (AZ-2) ──┼──→ Mount Target → EFS (Shared Files)
EC2 Instance (AZ-3) ──┘
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **File System** | The EFS resource (like a shared drive) |
| **Mount Target** | Network endpoint in each AZ to connect EC2 |
| **NFS Protocol** | Network File System protocol (port 2049) |
| **Access Point** | Application-specific entry point to EFS |
| **Throughput Mode** | Bursting or Provisioned throughput |
| **Performance Mode** | General Purpose or Max I/O |

---

## 🔄 EFS vs EBS vs S3

| Feature | EFS | EBS | S3 |
|---------|-----|-----|----|
| Type | File System (NFS) | Block Storage | Object Storage |
| Access | Multiple EC2 simultaneously | Single EC2 at a time | Via HTTP/API |
| Protocol | NFS | iSCSI/block | HTTP/REST |
| AZ | Multi-AZ | Single AZ | Regional |
| Scalability | Auto-scales | Fixed size | Unlimited |
| Use Case | Shared files, CMS | OS disk, DB | Backups, media |
| Cost | High (~$0.30/GB) | Medium (~$0.10/GB) | Low (~$0.023/GB) |

---

## ⚡ EFS Performance Modes

| Mode | Use Case |
|------|----------|
| **General Purpose** | Web servers, CMS, dev (default) |
| **Max I/O** | Big data, media processing, high parallelism |

## 💾 EFS Throughput Modes

| Mode | Description |
|------|-------------|
| **Bursting** | Auto-scales with storage size |
| **Provisioned** | Set fixed throughput regardless of size |
| **Elastic** | Auto-scales based on workload (recommended) |

## 🗃️ EFS Storage Classes

| Class | Description | Cost |
|-------|-------------|------|
| **Standard** | Frequently accessed data | Higher |
| **Standard-IA** | Infrequently accessed | Lower (30% savings) |
| **Archive** | Rarely accessed | Lowest |

---

## 🛠️ Hands-On: Create EFS & Mount on EC2

### Step 1: Create EFS File System
```
Services → EFS → Create File System
  → Customize
  → Name: my-efs
  → Storage Class: Standard
  → Performance Mode: General Purpose
  → Throughput Mode: Elastic
  → Next → Create
```

### Step 2: Configure Security Group (Allow NFS)
```
EC2 → Security Groups → Select your SG → Inbound Rules → Edit
  → Add Rule:
      Type: NFS
      Protocol: TCP
      Port: 2049
      Source: 0.0.0.0/0 (or your VPC CIDR)
  → Save Rules
```

### Step 3: Get Mount Target IP
```
EFS → File System → Network Tab
  → Copy the DNS name or IP of mount target in your AZ
  → Example: fs-12345678.efs.ap-south-1.amazonaws.com
```

### Step 4: Mount EFS on EC2
```bash
# Connect to EC2
sudo -i

# Update and install NFS client
apt update
apt install nfs-common -y

# Create mount point
mkdir -p /mnt/efs

# Mount EFS (use DNS name from console)
mount -t nfs4 -o nfsvers=4.1 fs-12345678.efs.ap-south-1.amazonaws.com:/ /mnt/efs

# Verify mount
df -h
```

### Step 5: Make Mount Permanent (Persist after reboot)
```bash
# Edit fstab
nano /etc/fstab

# Add this line at the bottom:
fs-12345678.efs.ap-south-1.amazonaws.com:/ /mnt/efs nfs4 defaults,_netdev 0 0

# Save (Ctrl+O, Enter, Ctrl+X)

# Reload systemd and remount
systemctl daemon-reload
mount -a

# Verify
df -h
```

### Step 6: Test Shared File System
```bash
# On EC2-1: Create files
cd /mnt/efs
touch abc{1..10}
ls

# On EC2-2 (same EFS mounted):
ls /mnt/efs    # You should see abc1..abc10!

# Clean up
rm -rf abc{1..5}
df -h
```

---

## 🔐 EFS Security

```
1. Security Groups → Allow NFS port 2049
2. IAM Policies → Control who can create/modify EFS
3. EFS Resource Policy → Fine-grained access control
4. Encryption at rest → AWS KMS
5. Encryption in transit → TLS (built into mount command)
```

### Mount with Encryption in Transit
```bash
# Install amazon-efs-utils
apt install amazon-efs-utils -y

# Mount with TLS
mount -t efs -o tls fs-12345678:/ /mnt/efs
```

---

## 🔁 EFS Lifecycle Management

Automatically move infrequently accessed files to cheaper storage:

```
EFS → File System → Storage Classes
  → Lifecycle Management: Enable
  → Transition to IA: After 30 days of no access
  → Transition out of IA: On first access
```

---

## ✅ Advantages

- **Multi-EC2 access** simultaneously
- **Fully managed** — no provisioning needed
- **Auto-scales** from 0 to petabytes
- **Multi-AZ** high availability
- **POSIX-compliant** file system
- **Lifecycle management** reduces costs

---

## ❌ Disadvantages

- **More expensive** than EBS or S3
- **Higher latency** than EBS
- **NFS protocol only** (not block-level access)
- **Linux only** — Windows not supported natively

---

## 🏆 Best Practices

- Use **EFS Access Points** for application isolation
- Enable **lifecycle management** to save costs
- Use **encryption in transit** (TLS) always
- Mount in **multiple AZs** for HA
- Use **EFS Elastic throughput** for unpredictable workloads
- Monitor with **CloudWatch EFS metrics**

---

## 📚 Use Cases

- **Content Management Systems** (WordPress shared media)
- **DevOps CI/CD** shared build artifacts
- **Data Science** shared datasets across instances
- **Web serving** shared static content
- **Container storage** (ECS/EKS persistent volumes)
