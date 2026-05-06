# 🖼️ AMI - Amazon Machine Image

## 📌 What is AMI?

An AMI (Amazon Machine Image) is a **pre-configured template** that contains the OS, application server, and applications needed to launch an EC2 instance. It's essentially a **snapshot/clone of an entire EC2 instance**.

> Think of AMI as a **master blueprint/template** — like a system image you can use to rapidly deploy identical servers.

---

## 🏗️ AMI Components

```
AMI
├── Root Volume Snapshot (OS + Data)
├── Launch Permissions (who can use this AMI)
├── Block Device Mappings (which volumes to attach)
└── Architecture (x86_64 or arm64)
```

---

## 📋 AMI Types

| Type | Description | Managed By |
|------|-------------|-----------|
| **AWS-provided** | Amazon Linux, Ubuntu, Windows, etc. | AWS |
| **AWS Marketplace** | Commercial AMIs with software | Third parties |
| **Community AMIs** | Public AMIs by community | Community |
| **Custom AMIs** | Your own AMIs from instances | You |

---

## 🔑 AMI Key Attributes

| Attribute | Description |
|-----------|-------------|
| **AMI ID** | ami-0123456789abcdef (region-specific) |
| **Region** | AMI is region-specific (must copy to share) |
| **Architecture** | x86_64 or arm64 (Graviton) |
| **Root Device** | EBS or Instance Store |
| **Virtualization** | HVM (Hardware Virtual Machine) — modern |
| **Platform** | Linux or Windows |

---

## 🛠️ Hands-On: Create Custom AMI

### Step 1: Launch & Configure EC2
```
EC2 → Launch Instance → Ubuntu 22.04 → Launch

Connect and set up:
sudo -i
apt update && apt upgrade -y
apt install nginx -y
echo "<h1>My Custom AMI Server</h1>" > /var/www/html/index.html
systemctl enable nginx
systemctl start nginx

# Install any software you need
apt install python3 python3-pip -y
pip3 install flask
```

### Step 2: Create AMI from Instance
```
EC2 → Instances → Select Instance
  → Actions → Image and Templates → Create Image
  → Image Name: my-nginx-ami-v1
  → Image Description: Ubuntu 22.04 with Nginx configured
  → No reboot: Uncheck (reboot ensures clean image)
  → Add Storage: default (8 GiB root)
  → Create Image

Wait 2-5 minutes for AMI to be available
```

### Step 3: Launch New Instance from Custom AMI
```
EC2 → AMIs → My AMIs → Select your AMI
  → Launch Instance from AMI
  → Instance Type: t2.micro
  → Configure → Launch

New instance has EVERYTHING from original — nginx, files, config!
```

---

## 🔗 Share AMI with Another AWS Account

### Method 1: Share via Console
```
EC2 → AMIs → My AMIs → Select AMI
  → Actions → Edit AMI Permissions
  → Add Account ID: 123456789012 (target account)
  → Add Permission → Save Changes

Target account:
  EC2 → AMIs → Shared with Me → Find your shared AMI → Launch
```

### Method 2: Make AMI Public
```
Actions → Edit AMI Permissions
  → Public → Save

⚠️ WARNING: Anyone can launch your AMI — remove sensitive data first!
```

---

## 📋 Copy AMI to Another Region

```
EC2 → AMIs → Select AMI
  → Actions → Copy AMI
  → Destination Region: us-east-1
  → Copy AMI

Reason: AMIs are region-specific. To use in another region, must copy.
```

---

## 📸 AMI vs Snapshot vs Launch Template

| Feature | AMI | Snapshot | Launch Template |
|---------|-----|----------|----------------|
| What | Full instance image | EBS volume backup | EC2 config blueprint |
| Contains | OS + data + config | Volume data only | Instance settings |
| Launches | New EC2 instances | New EBS volumes | New EC2 instances |
| Use case | Clone servers | Backup/restore data | Consistent launches |

---

## 🔄 AMI Lifecycle

```
Running EC2 → Create AMI → Available AMI
                                ↓
                        Launch new instances (anywhere)
                                ↓
                        Share with accounts/regions
                                ↓
                        Deregister (delete) when done
```

---

## 💰 AMI Costs

- Creating AMI: **Free**
- AMI itself: **Free** to store the registration
- **EBS Snapshots**: Charged ($0.05/GB/month) — the actual data storage
- Copying AMI: **Free** (but snapshot in target region is charged)

---

## ✅ Advantages

- **Rapid deployment** — Launch identical servers in seconds
- **Consistency** — All servers identical (no config drift)
- **Disaster Recovery** — Quickly restore from known-good state
- **Scale** — Auto Scaling uses AMI to launch instances
- **Sharing** — Share golden AMI across teams/accounts

## ❌ Disadvantages

- AMIs can become stale (need to update regularly)
- Large AMIs take longer to launch
- Region-specific — must copy for multi-region

## 🏆 Best Practices

- Create AMIs **after patching** and testing
- Use **semantic versioning**: my-app-ami-v1.0.0
- Set **deprecation date** on old AMIs
- Store AMI IDs in **parameter store** for automation
- Use **EC2 Image Builder** to automate AMI creation
- Remove **sensitive data** before sharing AMIs
- Tag AMIs with: Name, Version, Created-Date, Owner

---

## 📚 Related Services

- **EC2 Image Builder** — Automated AMI creation pipeline
- **Systems Manager** — Automate instance configuration before AMI
- **Auto Scaling** — Uses AMI via Launch Template
- **EBS Snapshots** — Underlying storage for AMI
