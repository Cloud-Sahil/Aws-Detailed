# 💾 EBS - Elastic Block Store + Snapshots

## 📌 What is EBS?

Amazon EBS (Elastic Block Store) provides **persistent block-level storage volumes** for use with EC2 instances. EBS volumes behave like **raw, unformatted block devices** — just like a hard drive you plug into your computer.

> Think of EBS as a **USB hard drive / SSD** that you can attach to an EC2 instance. When the instance stops, data is safe. You can detach and reattach to another instance.

---

## 🏗️ EBS Architecture

```
AWS AZ (ap-south-1a)
└── EC2 Instance
    ├── Root Volume (EBS) → /dev/xvda → / (OS)
    ├── Additional Volume 1 → /dev/xvdf → /data
    └── Additional Volume 2 → /dev/xvdg → /logs

Note: EBS volumes are LOCKED to one AZ
      (same AZ as EC2 instance)
```

---

## 📋 EBS Volume Types

| Type | Name | Use Case | Max IOPS | Max Throughput |
|------|------|----------|----------|----------------|
| **gp3** | General Purpose SSD (v3) | Default, most workloads | 16,000 | 1,000 MB/s |
| **gp2** | General Purpose SSD (v2) | Older default | 16,000 | 250 MB/s |
| **io1/io2** | Provisioned IOPS SSD | Critical DBs, low latency | 64,000 | 1,000 MB/s |
| **st1** | Throughput Optimized HDD | Big data, data warehouses | 500 | 500 MB/s |
| **sc1** | Cold HDD | Infrequently accessed | 250 | 250 MB/s |

> ✅ **Recommended:** Use **gp3** for most workloads — better performance and cheaper than gp2

---

## 🛠️ Hands-On: Create & Attach EBS Volume

### Step 1: Create Volume
```
EC2 → Volumes → Create Volume
  → Volume Type: gp3
  → Size: 10 GiB
  → Availability Zone: ap-south-1a (MUST match EC2 AZ!)
  → Create Volume
```

### Step 2: Attach to EC2
```
Volumes → Select Volume → Actions → Attach Volume
  → Instance: Select your EC2 instance
  → Device Name: /dev/sdf
  → Attach Volume
```

### Step 3: Format & Mount (inside EC2)
```bash
sudo -i

# List block devices
lsblk

# You'll see: xvdf (new disk, not mounted)

# Create partition
fdisk /dev/xvdf
  → n  (new partition)
  → p  (primary)
  → Enter (default partition number)
  → +10G (or Enter for full disk)
  → w  (write and exit)

# Format the partition
mkfs.ext4 /dev/xvdf1

# Check UUID
blkid /dev/xvdf1

# Mount
mount /dev/xvdf1 /mnt

# Verify
df -h
```

### Step 4: Persist Mount After Reboot
```bash
# Get UUID
blkid /dev/xvdf1
# Copy UUID

# Edit fstab
nano /etc/fstab

# Add line (replace UUID):
UUID=your-uuid-here  /mnt  ext4  defaults  0  2

# Test
umount /mnt
mount -a
df -h
```

---

## 📸 Snapshots

Snapshots are **point-in-time backups** of EBS volumes, stored in S3 (managed by AWS). They are **incremental** — only changed blocks are saved after the first snapshot.

```
EBS Volume → Snapshot (stored in S3)
           → Restore to new Volume (any AZ/Region)
           → Create AMI from Snapshot
```

### Incremental Snapshot Diagram
```
Snapshot 1 (full): [Block A] [Block B] [Block C]
Snapshot 2 (incr): [Block B changed] → only Block B saved
Snapshot 3 (incr): [Block C changed] → only Block C saved

All 3 snapshots are complete independently (AWS handles refs internally)
```

### Hands-On: Create Snapshot
```
EC2 → Volumes → Select Volume
  → Actions → Create Snapshot
  → Description: my-backup-2024-01
  → Create Snapshot
```

### Restore Snapshot to New Volume
```
EC2 → Snapshots → Select Snapshot
  → Actions → Create Volume from Snapshot
  → AZ: ap-south-1b (can be different AZ!)
  → Create Volume
```

### Copy Snapshot to Another Region
```
Snapshots → Select → Actions → Copy Snapshot
  → Destination Region: us-east-1
  → Copy
```

---

## 🔒 EBS Encryption

```
EBS → Create Volume → Enable Encryption
  → KMS Key: aws/ebs (default) or custom key
  → Create

Encrypts:
  - Data at rest
  - Data in transit between EC2 and EBS
  - All snapshots automatically
```

> You **CANNOT** encrypt an existing unencrypted volume directly.
> Workaround: Snapshot → Copy with Encryption → Create Volume → Attach

---

## 📊 EBS Multi-Attach (io1/io2 only)

```
io1/io2 volumes can be attached to multiple EC2 instances in SAME AZ
  → For clustered DBs, shared storage
  → Application must handle concurrent writes
```

---

## 🔄 EBS Lifecycle

```
[Create] → [Available] → [In-Use (attached)] → [Available] → [Deleted]
                ↓
         [Snapshot]
                ↓
         [Create New Volume]
```

---

## 🔢 EBS Size Limits

| Type | Min | Max |
|------|-----|-----|
| gp2/gp3 | 1 GiB | 16 TiB |
| io1 | 4 GiB | 16 TiB |
| io2 | 4 GiB | 64 TiB |
| st1 | 125 GiB | 16 TiB |
| sc1 | 125 GiB | 16 TiB |

---

## 💰 Pricing

| Action | Cost |
|--------|------|
| gp3 volume | ~$0.08/GB/month |
| Snapshot | ~$0.05/GB/month (only changed blocks) |
| Data transfer | Charged for cross-region copies |

---

## ✅ Advantages

- **Persistent** — Data survives EC2 stop/restart
- **High performance** — Low latency block storage
- **Flexible** — Multiple types for different workloads
- **Backup via Snapshots** — Point-in-time backup
- **Encryption** — KMS-based at no extra cost
- **Resize on the fly** — No downtime needed

---

## ❌ Disadvantages

- **AZ-locked** — Cannot access from different AZ without snapshot
- **Single instance** (except io1/io2 multi-attach)
- Snapshots can be slow for large volumes
- Not suitable for shared access (use EFS instead)

---

## 🏆 Best Practices

- Use **gp3** for most workloads (better than gp2)
- Enable **automatic snapshots** via Data Lifecycle Manager
- Use **io2** for mission-critical production databases
- Always **encrypt EBS volumes** (no performance overhead)
- **Delete unused volumes** to avoid unnecessary costs
- Monitor **VolumeQueueLength** in CloudWatch (should be near 0)
- Use **Snapshots** before making major changes

---

## 🔄 EBS vs Instance Store

| Feature | EBS | Instance Store |
|---------|-----|----------------|
| Persistence | Yes | No (lost on stop) |
| Performance | Good | Excellent (physically attached) |
| Backup | Snapshots | Manual (copy to S3) |
| Cost | Extra | Included in instance price |
| Use case | Production data | Cache, temp data |

---

## 📚 Related Services

- **EC2** — EBS is attached to EC2
- **Snapshots** → **AMI** — Create machine images from snapshots
- **Data Lifecycle Manager** — Automate snapshot policies
- **AWS Backup** — Centralized backup across services
