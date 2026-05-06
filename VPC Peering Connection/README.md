# 🔗 VPC Peering Connection

## 📌 What is VPC Peering?

VPC Peering is a networking connection between **two VPCs** that enables you to route traffic between them using **private IPv4 or IPv6 addresses**. Instances in either VPC can communicate as if they are within the same network.

> Think of VPC Peering as a **direct private tunnel** between two isolated networks — no internet, no VPN, just private connectivity.

---

## 🏗️ VPC Peering Architecture

```
AWS Account / Region

VPC-A: 192.168.0.0/16          VPC-B: 172.16.0.0/16
├── Subnet: 192.168.0.0/20     ├── Subnet: 172.16.0.0/20
│   └── EC2-A                  │   └── EC2-B
└── Route Table (VPC-A):       └── Route Table (VPC-B):
    172.16.0.0/16 → Peering        192.168.0.0/16 → Peering
           ↕                               ↕
    ←─────── Peering Connection ──────────→
          (pcx-0123456789abcdef)
```

---

## 📋 VPC Peering Key Facts

| Feature | Detail |
|---------|--------|
| **Transitive** | ❌ NOT supported (A-B, B-C ≠ A-C) |
| **Cross-Account** | ✅ Supported |
| **Cross-Region** | ✅ Supported (Inter-Region Peering) |
| **Overlapping CIDR** | ❌ NOT allowed (CIDRs must be unique) |
| **Cost** | Data transfer charges apply (cross-region higher) |
| **Bandwidth** | No bandwidth limit |
| **Encryption** | In-transit encrypted by default |

---

## ⚠️ Critical Rules

```
❌ No Transitive Peering:
   A ↔ B ↔ C  DOES NOT mean  A ↔ C
   You need separate peering A-C

❌ No Overlapping CIDRs:
   VPC-A: 10.0.0.0/16
   VPC-B: 10.0.0.0/16  ← CONFLICT! Can't peer

✅ Valid CIDRs for peering:
   VPC-A: 192.168.0.0/16
   VPC-B: 172.16.0.0/16
   VPC-C: 10.0.0.0/16
```

---

## 🛠️ Hands-On: Create VPC Peering (Same Account, Same Region)

### Step 1: Create VPC-A
```
VPC → Create VPC
  → VPC Only
  → Name: vpc-a
  → IPv4 CIDR: 192.168.0.0/16
  → Create
```

### Step 2: Create Subnet in VPC-A
```
Subnets → Create Subnet
  → VPC: vpc-a
  → Name: subnet-a
  → AZ: ap-south-1a
  → CIDR: 192.168.0.0/20
  → Create
```

### Step 3: Create VPC-B
```
VPC → Create VPC
  → Name: vpc-b
  → IPv4 CIDR: 172.16.0.0/16
  → Create
```

### Step 4: Create Subnet in VPC-B
```
Subnets → Create Subnet
  → VPC: vpc-b
  → Name: subnet-b
  → AZ: ap-south-1b
  → CIDR: 172.16.0.0/20
  → Create
```

### Step 5: Create Internet Gateways (for both VPCs)
```
For VPC-A:
  IGW → Create → Name: igw-a → Create
  Actions → Attach → vpc-a

For VPC-B:
  IGW → Create → Name: igw-b → Create
  Actions → Attach → vpc-b
```

### Step 6: Create Route Tables with Internet Access
```
For VPC-A:
  Route Tables → Create → Name: rt-a → VPC: vpc-a → Create
  Edit Routes → Add: 0.0.0.0/0 → igw-a → Save
  Edit Subnet Associations → subnet-a → Save

For VPC-B:
  Route Tables → Create → Name: rt-b → VPC: vpc-b → Create
  Edit Routes → Add: 0.0.0.0/0 → igw-b → Save
  Edit Subnet Associations → subnet-b → Save

Enable Auto-Assign Public IP on both subnets:
  Subnet → Actions → Edit Subnet Settings → Enable → Save
```

### Step 7: Create Peering Connection
```
VPC → Peering Connections → Create Peering Connection
  → Name: vpc-a-to-vpc-b
  → VPC ID (Requester): vpc-a
  → Account: My Account
  → Region: This Region
  → VPC ID (Accepter): vpc-b
  → Create Peering Connection

Status: Pending Acceptance
```

### Step 8: Accept Peering Request (VPC-B side)
```
Peering Connections → Select pcx-xxx (Pending Acceptance)
  → Actions → Accept Request → Accept
Status: Active ✅
```

### Step 9: Update Route Tables (BOTH VPCS — Critical!)
```
VPC-A Route Table (rt-a):
  → Edit Routes → Add Route:
    Destination: 172.16.0.0/16
    Target: Peering Connection → pcx-xxx
  → Save

VPC-B Route Table (rt-b):
  → Edit Routes → Add Route:
    Destination: 192.168.0.0/16
    Target: Peering Connection → pcx-xxx
  → Save
```

### Step 10: Launch EC2 in Both VPCs
```
EC2-A in vpc-a subnet-a (with Public IP)
EC2-B in vpc-b subnet-b (with Public IP)
```

### Step 11: Configure Security Groups
```
EC2-A Security Group → Inbound:
  SSH (22) - 0.0.0.0/0
  ICMP (All) - 172.16.0.0/16 (allow ping from VPC-B)

EC2-B Security Group → Inbound:
  SSH (22) - 0.0.0.0/0
  ICMP (All) - 192.168.0.0/16 (allow ping from VPC-A)
```

### Step 12: Test Peering
```bash
# Connect to EC2-A via SSH
ssh -i key.pem ubuntu@<EC2-A public IP>

# Ping EC2-B using its PRIVATE IP
ping 172.16.0.x   # EC2-B's private IP

# Should work! ICMP replies confirm peering is working ✅
```

---

## 🌍 Cross-Account VPC Peering

```
Account A: Creates peering request → enters Account B's VPC ID
Account B: Accepts request in their console
Both: Update route tables and security groups
```

---

## 🔄 VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|---------|------------|----------------|
| Connections | Point-to-point | Hub-and-spoke |
| Transitive | No | Yes |
| Max VPCs | Limited (mesh complexity) | Thousands |
| Cost | Data transfer only | $0.05/hour + data |
| Use case | Few VPCs | Many VPCs |

> For 3+ VPCs that need to communicate → use **Transit Gateway**

---

## ✅ Advantages

- **Private connectivity** — no internet
- **Low latency** — direct connection
- **No single point of failure**
- **Cross-account & cross-region** support
- **No bandwidth limit**

## ❌ Disadvantages

- No transitive routing
- CIDR ranges can't overlap
- Management complexity grows with many VPCs
- Need to update route tables on BOTH sides

## 🏆 Best Practices

- Plan **CIDR ranges** carefully before creating VPCs
- Use **Transit Gateway** when you have many VPCs
- Use **VPC Flow Logs** to monitor peering traffic
- Restrict **security groups** to only needed CIDRs
- Tag peering connections clearly (name, purpose, accounts)
