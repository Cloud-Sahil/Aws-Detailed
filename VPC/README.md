# 🌐 VPC - Virtual Private Cloud

## 📌 What is VPC?

Amazon VPC (Virtual Private Cloud) lets you launch AWS resources in a **logically isolated virtual network** that you define. You have complete control over your virtual networking environment.

> Think of VPC as your **own private data center inside AWS** — with your own IP ranges, subnets, routing, and firewall rules.

---

## 🏗️ VPC Architecture

```
AWS Region (ap-south-1)
└── VPC: 192.168.0.0/16
    ├── Public Subnet: 192.168.0.0/20  (AZ: ap-south-1a)
    │   ├── EC2 Instance (nginx)
    │   └── NAT Gateway
    ├── Private Subnet: 192.168.16.0/20 (AZ: ap-south-1b)
    │   └── EC2 Instance (database)
    ├── Internet Gateway (IGW) ← attached to VPC
    ├── Route Table (Public) → 0.0.0.0/0 → IGW
    ├── Route Table (Private) → 0.0.0.0/0 → NAT GW
    └── Security Groups (per instance)
```

---

## 📋 Key Components

| Component | Description |
|-----------|-------------|
| **VPC** | Your isolated virtual network with CIDR block |
| **Subnet** | Segment of VPC in a specific AZ (public or private) |
| **Internet Gateway (IGW)** | Allows internet access to/from public subnets |
| **NAT Gateway** | Lets private subnet instances access internet (outbound only) |
| **Route Table** | Rules that control where traffic is directed |
| **Security Group** | Instance-level stateful firewall (Allow only) |
| **NACL** | Subnet-level stateless firewall (Allow + Deny) |
| **Elastic IP** | Static public IP |
| **VPC Peering** | Connect two VPCs privately |
| **VPN Gateway** | Connect on-premise to AWS |

---

## 🔢 CIDR Block Reference

| CIDR | IP Range | Hosts | Use Case |
|------|----------|-------|----------|
| /16 | x.x.0.0 - x.x.255.255 | 65,536 | Large VPC |
| /20 | x.x.x.0 - x.x.x+15.255 | 4,096 | Subnet |
| /24 | x.x.x.0 - x.x.x.255 | 256 | Small subnet |
| /28 | x.x.x.0 - x.x.x.15 | 16 | Tiny subnet |

> AWS reserves **5 IPs** per subnet (first 4 + last 1)

---

## 🛠️ Hands-On: Create VPC from Scratch

### Step 1: Create VPC
```
VPC → Your VPCs → Create VPC
  → VPC Only
  → Name: my-vpc
  → IPv4 CIDR: 192.168.0.0/16
  → Create VPC
```

### Step 2: Create Subnets
```
VPC → Subnets → Create Subnet
  → VPC: my-vpc
  → Subnet Name: public-subnet-1a
  → AZ: ap-south-1a
  → IPv4 CIDR: 192.168.0.0/20
  → Create

Create another:
  → Subnet Name: private-subnet-1b
  → AZ: ap-south-1b
  → IPv4 CIDR: 192.168.16.0/20
```

### Step 3: Enable Auto-Assign Public IP (Public Subnet)
```
Subnets → Select public-subnet → Actions
  → Edit Subnet Settings → Enable Auto-assign Public IPv4 → Save
```

### Step 4: Create Internet Gateway
```
VPC → Internet Gateways → Create
  → Name: my-igw → Create
  → Actions → Attach to VPC → Select my-vpc → Attach
```

### Step 5: Create Route Table (Public)
```
VPC → Route Tables → Create Route Table
  → Name: public-rt
  → VPC: my-vpc
  → Create

Actions → Edit Routes → Add Route:
  → Destination: 0.0.0.0/0
  → Target: Internet Gateway → my-igw
  → Save

Actions → Edit Subnet Associations:
  → Select public-subnet → Save
```

### Step 6: Launch EC2 in VPC
```
EC2 → Launch Instance
  → Network Settings → Edit
  → VPC: my-vpc
  → Subnet: public-subnet-1a
  → Auto-assign Public IP: Enable
  → Launch
```

### Step 7: Verify
```bash
# Connect to instance
ssh -i key.pem ubuntu@<public-ip>
sudo apt update   # Should work if IGW is configured correctly
```

---

## 🔀 NAT Gateway

NAT Gateway allows instances in **private subnets** to access the internet (for updates, APIs) but **prevents** internet from initiating connections to them.

```
Private EC2 → Private Subnet → NAT Gateway (in Public Subnet) → IGW → Internet
```

### Hands-On: Create NAT Gateway
```
VPC → NAT Gateways → Create NAT Gateway
  → Name: my-nat
  → Subnet: public-subnet (NAT MUST be in public subnet!)
  → Connectivity Type: Public
  → Allocate Elastic IP → Create

Route Table (Private):
  → Create new route table for private subnet
  → Edit Routes → Add: 0.0.0.0/0 → NAT Gateway
  → Edit Subnet Associations → Add private subnet
```

---

## 🔄 Public vs Private Subnet

| Feature | Public Subnet | Private Subnet |
|---------|---------------|----------------|
| Internet access | Direct via IGW | Via NAT Gateway only |
| Public IP | Yes (auto-assign) | No |
| Use case | Web servers, ALB | Databases, backend |
| Route | 0.0.0.0/0 → IGW | 0.0.0.0/0 → NAT GW |

---

## 🔗 VPC Peering

Connect two VPCs (same or different region/account) using **private IPs**.

```
VPC-A (192.168.0.0/16) ←→ Peering Connection ←→ VPC-B (172.16.0.0/16)
```

### Hands-On: VPC Peering
```
Step 1: Create VPC-A (192.168.0.0/16) with subnet + EC2
Step 2: Create VPC-B (172.16.0.0/16) with subnet + EC2

Step 3: Create Peering Connection (from VPC-A)
  VPC → Peering Connections → Create
  → Requester: VPC-A
  → Accepter: VPC-B (same account/region or different)
  → Create

Step 4: Accept Request (from VPC-B side)
  Peering Connections → Select → Actions → Accept Request

Step 5: Update Route Tables (BOTH VPCs)
  VPC-A Route Table:
    Add: 172.16.0.0/16 → Peering Connection
  VPC-B Route Table:
    Add: 192.168.0.0/16 → Peering Connection

Step 6: Test
  # From VPC-A EC2:
  ping <VPC-B private IP>
```

### VPC Peering Limitations
- **No transitive peering** (A→B→C doesn't allow A→C)
- CIDR blocks **must NOT overlap**
- Works across accounts and regions

---

## 🛡️ Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| Level | Instance | Subnet |
| Stateful | ✅ Yes | ❌ No |
| Rules | Allow only | Allow + Deny |
| Rule Order | All rules checked | Numbered order |
| Default | Deny all inbound | Allow all |

---

## 🗺️ VPC Resource Map

```
VPC Console → Your VPC → Resource Map Tab
  Shows visual diagram of: VPC → Subnets → Route Tables → IGW → NAT
```

---

## ✅ Advantages

- Complete network control
- Isolated and secure environment
- Custom IP address ranges
- Multiple layers of security
- Integration with all AWS services

---

## ❌ Disadvantages

- Complex to set up initially
- Misconfiguration can lead to security issues
- VPC peering doesn't support transitive routing
- NAT Gateway has additional cost

---

## 🏆 Best Practices

- Always use **multi-AZ** for high availability
- Keep **databases in private subnets**
- Use **Security Groups** at instance level, **NACLs** at subnet level
- Use **VPC Flow Logs** to monitor traffic
- Use **separate VPCs** for dev/staging/prod
- Use **Transit Gateway** instead of mesh peering for many VPCs
- Enable **DNS hostnames and resolution** in VPC settings

---

## 📚 Related Services

- **Direct Connect** — Dedicated connection from on-premise
- **VPN Gateway** — Encrypted tunnel to on-premise
- **Transit Gateway** — Hub-and-spoke VPC connectivity
- **PrivateLink** — Private access to AWS services
- **Route 53** — DNS within VPC (private hosted zones)
