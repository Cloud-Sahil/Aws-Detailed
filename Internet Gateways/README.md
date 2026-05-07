# 🌐 Internet Gateway (IGW) - Complete Guide

## 📌 What is an Internet Gateway?

An **Internet Gateway (IGW)** is a horizontally scaled, redundant, and highly available VPC component that allows communication between resources in your VPC and the public internet.

> Think of IGW as the **main gate of your building (VPC)** — everything entering or leaving through the internet must pass through this gate.

---

## 🧠 Simple Analogy

```
Your House (VPC)
    │
  Main Gate (IGW)  ← Only 1 gate per house
    │
  Road (Internet)
```

- **No gate** = no internet access at all
- **Gate exists but no road sign (route)** = still no internet
- **Gate + Road sign (route table entry)** = ✅ internet works

---

## 🏗️ IGW Architecture

```
                        INTERNET
                            │
                            │
                   ┌────────▼─────────┐
                   │  Internet Gateway │  ← Attached to VPC
                   │   (igw-xxxxxxxx)  │  ← FREE, fully managed
                   └────────┬─────────┘
                            │
              ┌─────────────▼──────────────┐
              │         VPC: 10.0.0.0/16   │
              │                            │
              │  Public Subnet             │
              │  ┌──────────────────────┐  │
              │  │ Route Table:         │  │
              │  │  10.0.0.0/16 → local │  │
              │  │  0.0.0.0/0  → IGW ✅ │  │
              │  │                      │  │
              │  │  [EC2 Web Server]    │  │
              │  │  Private IP: 10.0.0.5│  │
              │  │  Public  IP: 52.x.x.x│  │
              │  └──────────────────────┘  │
              └────────────────────────────┘
```

---

## 🔑 Key Facts About IGW

| Property | Detail |
|----------|--------|
| **Per VPC limit** | Only **ONE** IGW can be attached per VPC |
| **Redundancy** | Fully redundant by AWS — no single point of failure |
| **Scaling** | Horizontally auto-scales — no bandwidth limit |
| **Cost** | **FREE** — no charge for IGW itself |
| **Direction** | **Bidirectional** — inbound AND outbound |
| **IPv4 + IPv6** | Supports both |
| **State** | Detached (default) → must attach to VPC |

---

## 🔄 How IGW Works — Packet Flow

### Outbound (EC2 → Internet)
```
Step 1: EC2 sends packet
        Source IP:      10.0.0.5  (private IP)
        Destination IP: 8.8.8.8   (Google DNS)

Step 2: Route Table checks
        Destination 8.8.8.8 → matches 0.0.0.0/0 → IGW

Step 3: IGW performs NAT (Network Address Translation)
        Replaces: Source IP 10.0.0.5
        With:     Elastic/Public IP 52.10.20.30

Step 4: Packet sent to internet
        Internet sees: Source = 52.10.20.30, Dest = 8.8.8.8
```

### Inbound (Internet → EC2)
```
Step 1: Internet sends packet
        Source IP:      1.2.3.4
        Destination IP: 52.10.20.30 (public IP of EC2)

Step 2: IGW receives packet
        Translates: 52.10.20.30 → 10.0.0.5 (private IP)

Step 3: Routes to EC2 inside VPC
        EC2 receives packet ✅
```

### IGW NAT Translation Diagram
```
Inside VPC                    │                 Internet
                              │
EC2: 10.0.0.5 ──────────────→│←── IGW translates ──→ 52.10.20.30
(Private IP)                  │                      (Public/Elastic IP)
                              │
          IGW does 1:1 NAT mapping
```

---

## 🛠️ Hands-On: Create and Configure IGW

### Step 1: Create Internet Gateway
```
AWS Console → VPC → Internet Gateways
  → Create Internet Gateway
      Name tag: my-igw
  → Create Internet Gateway

Status shown: ⚠️ Detached
```

### Step 2: Attach IGW to VPC
```
Internet Gateways → Select: my-igw
  → Actions → Attach to VPC
  → Available VPCs: select your VPC
  → Attach Internet Gateway

Status shown: ✅ Attached
```

### Step 3: Add Route in Route Table
```
⚠️ CRITICAL STEP — IGW alone does NOT route traffic!

VPC → Route Tables → Select public subnet's route table
  → Routes tab → Edit Routes
  → Add Route:
      Destination: 0.0.0.0/0
      Target: Internet Gateway → igw-xxxxxxxx
  → Save Changes
```

### Step 4: Enable Public IP on Subnet
```
VPC → Subnets → Select public subnet
  → Actions → Edit Subnet Settings
  → ✅ Enable auto-assign public IPv4 address
  → Save

✅ Now any EC2 launched here gets a public IP automatically
```

### Step 5: Verify with EC2
```bash
# Launch EC2 in public subnet
# Connect via SSH

# Test internet access
ping 8.8.8.8
curl -I https://google.com

# See your public IP (should be Elastic IP or auto-assigned)
curl ifconfig.me
```

---

## ⚠️ Common Mistakes & Fixes

### Mistake 1: IGW attached but internet not working
```
Problem: Route table doesn't have entry for IGW
Fix:     Add route: 0.0.0.0/0 → igw-xxx in route table
```

### Mistake 2: Route table has IGW but still no internet
```
Problem: Route table not associated with the subnet
Fix:
  Route Tables → Select rt → Subnet Associations
  → Edit → Check your public subnet → Save
```

### Mistake 3: EC2 has no public IP
```
Problem: Instance has no public/elastic IP
         Even with IGW + route, needs a public IP
Fix Option 1: Enable auto-assign public IP on subnet
Fix Option 2: Allocate and associate an Elastic IP
```

### Mistake 4: Security Group blocking traffic
```
Problem: IGW is fine but SG rejects the traffic
Fix:
  EC2 → Security Groups → Inbound Rules
  Add: HTTP (80) from 0.0.0.0/0
  Add: HTTPS (443) from 0.0.0.0/0
  Add: SSH (22) from your IP
```

### Mistake 5: Trying to attach 2 IGWs to one VPC
```
Problem: Error — "VPC already has an internet gateway attached"
Rule:    ONE IGW per VPC maximum
Fix:     Use the existing IGW
```

---

## 🔄 IGW vs Other Gateways

| Gateway | Connects | Direction | Cost |
|---------|---------|-----------|------|
| **Internet Gateway** | VPC ↔ Public Internet | Bidirectional | Free |
| **NAT Gateway** | Private Subnet → Internet | Outbound only | ~$32/month |
| **Virtual Private Gateway** | VPC ↔ On-premise (VPN) | Bidirectional | VPN charges |
| **Direct Connect** | VPC ↔ On-premise (fiber) | Bidirectional | High |
| **Egress-Only IGW** | VPC → Internet (IPv6 only) | Outbound only | Free |

---

## 📋 IGW Checklist

Before saying "internet doesn't work", check ALL of these:

```
□ IGW created?
□ IGW attached to correct VPC? (Status = Attached)
□ Route table has: 0.0.0.0/0 → igw-xxx?
□ Route table associated with the correct SUBNET?
□ EC2 instance has a PUBLIC IP or ELASTIC IP?
□ Security Group allows inbound traffic on required port?
□ NACL (if custom) allows traffic in both directions?
□ EC2 is in the PUBLIC subnet (not private)?
```

---

## 🔐 Security Considerations

```
IGW itself has NO security — it just enables routing.
Security is handled by:

1. Security Groups     → Instance-level firewall
2. NACLs              → Subnet-level firewall  
3. WAF                → Web Application Firewall (HTTP attacks)
4. Shield             → DDoS protection
5. Elastic IP         → Control which IPs are exposed

Best Practice:
  ✅ Only web-facing servers in public subnet with IGW
  ✅ Keep databases/backend in private subnets (no IGW route)
  ✅ Use Security Groups to restrict inbound ports
```

---

## ✅ Advantages

- **Free** — no cost for the gateway itself
- **Fully managed** — no patching or maintenance
- **Highly available** — redundant across AZs automatically
- **No bandwidth limit** — scales automatically
- **Supports IPv4 and IPv6**
- **Simple to configure** — create, attach, add route

## ❌ Disadvantages

- **Only one per VPC** — cannot have multiple IGWs
- **No filtering** — IGW doesn't inspect or filter traffic (use SG/NACL/WAF)
- **Public exposure** — resources with public IPs are internet-accessible (manage with SGs)

---

## 🏆 Best Practices

- Always use **explicit route tables** — don't rely on the Main route table
- Only put **load balancers and bastion hosts** in public subnets (not DB, app servers)
- Use **Elastic IPs** for servers that need a fixed public IP
- Tag IGW clearly: `Name: prod-vpc-igw`
- Use **VPC Flow Logs** to monitor all traffic through IGW
- For **IPv6**, use Egress-Only IGW for private IPv6 outbound traffic

---

## 📚 Related Files

- [03_VPC.md](03_VPC.md) — VPC setup
- [20_Route_Tables.md](20_Route_Tables.md) — Route table configuration
- [21_NAT_Gateway.md](21_NAT_Gateway.md) — Private subnet internet access
- [07_ALB.md](07_ALB.md) — Load balancer in public subnet
