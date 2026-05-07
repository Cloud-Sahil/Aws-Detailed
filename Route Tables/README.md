# 🗺️ Route Tables - Complete Guide

## 📌 What is a Route Table?

A **Route Table** is a set of rules (called routes) that determines where network traffic is directed within a VPC. Every subnet in your VPC must be associated with a route table, which controls the routing for that subnet.

> Think of a Route Table as a **GPS / Navigation System** for your VPC — when a packet arrives, it asks the route table: *"Where should I send this?"*

---

## 🧠 Simple Analogy

```
You're at a Junction (Subnet)
    ↓
You look at the Sign Board (Route Table)
    ├── "To Mumbai (10.0.0.0/16)" → Take local road
    ├── "To Internet (0.0.0.0/0)" → Take Highway via IGW
    └── "To Office VPC (172.16.0.0/16)" → Take VPC Peering road

The Sign Board = Route Table
The Directions = Routes
```

---

## 🏗️ Route Table Architecture

```
VPC: 10.0.0.0/16
│
├── Public Route Table (public-rt)
│   ├── Subnet: public-subnet-1a  ← ASSOCIATED
│   ├── Subnet: public-subnet-1b  ← ASSOCIATED
│   └── Routes:
│       ┌────────────────────┬─────────────────┐
│       │ Destination        │ Target          │
│       ├────────────────────┼─────────────────┤
│       │ 10.0.0.0/16       │ local           │ ← VPC internal
│       │ 0.0.0.0/0         │ igw-xxxxxx      │ ← Internet
│       └────────────────────┴─────────────────┘
│
├── Private Route Table (private-rt)
│   ├── Subnet: private-subnet-1a ← ASSOCIATED
│   ├── Subnet: private-subnet-1b ← ASSOCIATED
│   └── Routes:
│       ┌────────────────────┬─────────────────┐
│       │ Destination        │ Target          │
│       ├────────────────────┼─────────────────┤
│       │ 10.0.0.0/16       │ local           │ ← VPC internal
│       │ 0.0.0.0/0         │ nat-xxxxxx      │ ← NAT Gateway
│       └────────────────────┴─────────────────┘
│
└── Main Route Table (default)
    ├── Subnets not explicitly associated use this
    └── Routes:
        ┌────────────────────┬─────────────────┐
        │ Destination        │ Target          │
        ├────────────────────┼─────────────────┤
        │ 10.0.0.0/16       │ local           │
        └────────────────────┴─────────────────┘
```

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **Route** | A single rule: if destination matches → send to target |
| **Destination** | IP range (CIDR) the rule applies to |
| **Target** | Where to send the packet (IGW, NAT, peer, etc.) |
| **Local Route** | Auto-added for VPC CIDR — cannot delete |
| **Main Route Table** | Default for subnets with no explicit association |
| **Custom Route Table** | Created by you for specific subnets |
| **Subnet Association** | Linking a subnet to a specific route table |

---

## 📋 Route Table — Full Structure Explained

```
Route Table: public-rt
ID: rtb-0abc12345
VPC: vpc-xxxxxxxx
│
├── Routes (the actual rules)
│   ┌─────────────────────┬──────────────────┬────────────┬──────────┐
│   │ Destination         │ Target           │ Status     │ Origin   │
│   ├─────────────────────┼──────────────────┼────────────┼──────────┤
│   │ 10.0.0.0/16        │ local            │ Active     │ Default  │
│   │ 0.0.0.0/0          │ igw-0abc12345    │ Active     │ Custom   │
│   │ 172.16.0.0/16      │ pcx-0def67890    │ Active     │ Custom   │
│   │ pl-63a5400a        │ vpce-0xyz789     │ Active     │ Custom   │
│   └─────────────────────┴──────────────────┴────────────┴──────────┘
│
└── Subnet Associations
    ├── public-subnet-1a (explicit)
    └── public-subnet-1b (explicit)
```

---

## 🎯 Route Priority — Longest Prefix Match

When a packet's destination matches **multiple routes**, the **most specific (longest prefix)** route wins.

```
Example Route Table:
  10.0.0.0/16    → local          (matches 65,536 IPs)
  10.0.1.0/24   → eni-12345      (matches 256 IPs — MORE SPECIFIC)
  0.0.0.0/0     → igw-xxxxx      (matches ALL IPs)

Packet going to 10.0.1.5:
  ✅ Matches: 10.0.0.0/16  (but not most specific)
  ✅ Matches: 10.0.1.0/24  ← WINNER (longest prefix /24 > /16)
  ✅ Matches: 0.0.0.0/0    (least specific)
  → Sent to: eni-12345

Packet going to 10.0.5.10:
  ✅ Matches: 10.0.0.0/16  ← WINNER (no /24 for this range)
  ✅ Matches: 0.0.0.0/0
  → Sent to: local

Packet going to 8.8.8.8:
  ✅ Matches: 0.0.0.0/0 only ← WINNER (only match)
  → Sent to: igw-xxxxx
```

---

## 🔀 All Possible Route Targets

| Target Type | Used For | Example |
|------------|---------|---------|
| `local` | Traffic within VPC | Auto-added, always present |
| `igw-xxx` | Public internet access | 0.0.0.0/0 → igw |
| `nat-xxx` | Private → internet (outbound) | 0.0.0.0/0 → nat |
| `vgw-xxx` | VPN to on-premise | 192.168.0.0/16 → vgw |
| `pcx-xxx` | VPC Peering traffic | 172.16.0.0/16 → pcx |
| `tgw-xxx` | Transit Gateway (multi-VPC) | 10.0.0.0/8 → tgw |
| `vpce-xxx` | VPC Endpoint (S3, DynamoDB) | pl-xxxxx → vpce |
| `eigw-xxx` | Egress-Only IGW (IPv6) | ::/0 → eigw |
| `eni-xxx` | Specific Network Interface | 10.0.5.0/24 → eni |
| `i-xxx` | Specific EC2 instance | (NAT instance) |

---

## 🛠️ Hands-On: Complete Route Table Setup

### Scenario: 2-tier app (Public web + Private DB)

```
VPC: 10.0.0.0/16
  ├── Public Subnet:  10.0.1.0/24 (web servers)
  └── Private Subnet: 10.0.2.0/24 (databases)
```

### Step 1: Create Public Route Table
```
VPC → Route Tables → Create Route Table
  → Name: public-rt
  → VPC: my-vpc
  → Create Route Table
```

### Step 2: Add Internet Route to Public RT
```
Select public-rt
  → Routes tab → Edit Routes → Add Route
      Destination: 0.0.0.0/0
      Target: Internet Gateway → igw-xxxxxxxx
  → Save Changes

Result:
  10.0.0.0/16 → local     (auto-added)
  0.0.0.0/0   → igw-xxx   (just added)
```

### Step 3: Associate Public Subnet with Public RT
```
Select public-rt
  → Subnet Associations tab → Edit Subnet Associations
  → ✅ Check: public-subnet (10.0.1.0/24)
  → Save Associations
```

### Step 4: Create Private Route Table
```
VPC → Route Tables → Create Route Table
  → Name: private-rt
  → VPC: my-vpc
  → Create Route Table
```

### Step 5: Add NAT Gateway Route to Private RT
```
Select private-rt
  → Routes tab → Edit Routes → Add Route
      Destination: 0.0.0.0/0
      Target: NAT Gateway → nat-xxxxxxxx
  → Save Changes

Result:
  10.0.0.0/16 → local     (auto-added)
  0.0.0.0/0   → nat-xxx   (just added)
```

### Step 6: Associate Private Subnet with Private RT
```
Select private-rt
  → Subnet Associations tab → Edit Subnet Associations
  → ✅ Check: private-subnet (10.0.2.0/24)
  → Save Associations
```

### Step 7: Verify via Resource Map
```
VPC → Your VPCs → Select VPC → Resource Map tab

Visual shows:
  public-subnet → public-rt → igw
  private-subnet → private-rt → nat → igw
```

---

## 🏗️ Route Table Design Patterns

### Pattern 1: Simple — Everything Public
```
One Route Table for all subnets:
  10.0.0.0/16 → local
  0.0.0.0/0   → igw-xxx

✅ Use for: Quick dev/test environments
❌ Not for: Production (security risk)
```

### Pattern 2: Standard — Public + Private (Recommended)
```
Public RT:
  10.0.0.0/16 → local
  0.0.0.0/0   → igw-xxx    ← Direct internet

Private RT:
  10.0.0.0/16 → local
  0.0.0.0/0   → nat-xxx    ← Outbound internet via NAT

✅ Use for: Most production apps
```

### Pattern 3: 3-Tier — Public + Private + Isolated
```
Public RT:
  10.0.0.0/16 → local
  0.0.0.0/0   → igw-xxx

Private RT (App Tier):
  10.0.0.0/16 → local
  0.0.0.0/0   → nat-xxx

Isolated RT (DB Tier):
  10.0.0.0/16 → local
  (NO internet route — completely isolated!)

✅ Use for: Banks, healthcare, high-security apps
```

### Pattern 4: Multi-VPC with Peering
```
VPC-A Public RT:
  10.0.0.0/16  → local
  0.0.0.0/0    → igw-xxx
  172.16.0.0/16 → pcx-xxx  ← Route to VPC-B

VPC-B Private RT:
  172.16.0.0/16 → local
  10.0.0.0/16   → pcx-xxx  ← Route to VPC-A

✅ Use for: Microservices across VPCs
```

### Pattern 5: On-Premise Connection (VPN/Direct Connect)
```
Private RT:
  10.0.0.0/16   → local
  0.0.0.0/0     → nat-xxx
  192.168.0.0/16 → vgw-xxx  ← On-premise network via VPN

✅ Use for: Hybrid cloud architectures
```

---

## 🔍 Main Route Table vs Custom Route Table

| Feature | Main Route Table | Custom Route Table |
|---------|-----------------|-------------------|
| Created by | AWS automatically | You create it |
| Default for | All subnets with no explicit association | Only explicitly associated subnets |
| Can delete? | No | Yes (if no associations) |
| Best practice | Leave as-is (only local route) | Use for all subnets |
| Risk | Modifying it affects ALL unassociated subnets | Isolated — only affects linked subnets |

> ⚠️ **Best Practice:** Never modify the Main route table. Create custom route tables for all subnets.

---

## 📊 Route Table Limits

| Limit | Default | Max (request increase) |
|-------|---------|----------------------|
| Route tables per VPC | 200 | 200 |
| Routes per route table | 50 | 1,000 |
| Subnet associations per RT | Unlimited | — |

---

## ⚠️ Troubleshooting Route Issues

### Issue: Traffic not reaching internet from public subnet
```
Check:
1. Route table has 0.0.0.0/0 → igw-xxx?
   VPC → Route Tables → Routes tab

2. This route table is associated with the subnet?
   Route Tables → Subnet Associations

3. IGW is attached to VPC?
   VPC → Internet Gateways → Status = Attached

4. EC2 has public IP?
   EC2 → Instance → Public IPv4 address (not empty)
```

### Issue: Private subnet can't reach internet for updates
```
Check:
1. Route table has 0.0.0.0/0 → nat-xxx?
2. NAT Gateway is in PUBLIC subnet (not private!)?
3. NAT Gateway status = Available?
4. Public subnet's route table has 0.0.0.0/0 → igw-xxx?
   (NAT needs this to forward traffic to internet)
```

### Issue: VPC Peering connected but ping fails
```
Check:
1. Route added in VPC-A RT pointing to VPC-B CIDR via pcx?
2. Route added in VPC-B RT pointing to VPC-A CIDR via pcx?
   (BOTH sides must have routes — bidirectional!)
3. Security Groups allow ICMP from other VPC's CIDR?
```

---

## ✅ Advantages

- **Granular control** over traffic flow per subnet
- **Multiple patterns** — public, private, isolated, peering
- **Free** — no charges for route tables
- **Flexible** — add/remove routes instantly without downtime
- **Supports all target types** — IGW, NAT, VPN, Peering, TGW

## ❌ Disadvantages

- Easy to misconfigure (missing route = no connectivity)
- Must update BOTH sides for VPC Peering routes
- No automatic cleanup if target (NAT, IGW) is deleted

---

## 🏆 Best Practices

- Create **separate route tables** for each tier (public, private, isolated)
- **Never modify** the Main route table
- Use **descriptive names**: `prod-public-rt`, `prod-private-rt-az1`
- For **HA**, create separate private route table per AZ (each with own NAT GW)
- Add **VPC Endpoints** for S3/DynamoDB to avoid routing through NAT (saves cost)
- Regularly **audit routes** — remove unused/stale entries
- Use **VPC Resource Map** to visually verify all associations

---

## 📚 Related Files

- [03_VPC.md](03_VPC.md) — VPC fundamentals
- [19_Internet_Gateway.md](19_Internet_Gateway.md) — IGW setup
- [21_NAT_Gateway.md](21_NAT_Gateway.md) — NAT Gateway setup
- [13_Peering_Connection.md](13_Peering_Connection.md) — VPC Peering routes
