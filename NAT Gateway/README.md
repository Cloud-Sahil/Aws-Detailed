# 🔒 NAT Gateway - Complete Guide

## 📌 What is a NAT Gateway?

A **NAT Gateway (Network Address Translation Gateway)** is a fully managed AWS service that allows instances in a **private subnet** to initiate outbound connections to the internet while **preventing the internet from initiating inbound connections** to those instances.

> Think of NAT Gateway as a **one-way glass / secret passage** — your private servers can look out and reach the internet (for updates, APIs), but the internet cannot look in or reach your private servers.

---

## 🧠 Simple Analogy

```
Private Server (wants to download updates)
        │
        ▼
  NAT Gateway  ←── Only outbound allowed
  (Like a proxy)
        │
        ▼
    Internet ──✖── Cannot initiate connection BACK to private server
```

**Real-life analogy:** Like calling someone from a payphone.
- You (private server) can call out to internet
- But the caller ID shows payphone number (NAT's IP), not yours
- No one can call YOU back directly on the payphone

---

## 🏗️ NAT Gateway Architecture

```
                      INTERNET
                          │
                          ▼
               ┌─────────────────┐
               │ Internet Gateway │
               └────────┬────────┘
                        │
        ┌───────────────▼──────────────────┐
        │            VPC: 10.0.0.0/16      │
        │                                   │
        │  ┌──── Public Subnet ──────────┐  │
        │  │  10.0.1.0/24               │  │
        │  │                            │  │
        │  │  ┌─────────────────────┐   │  │
        │  │  │   NAT GATEWAY       │   │  │ ← MUST be in PUBLIC subnet
        │  │  │   nat-0abc12345     │   │  │
        │  │  │   EIP: 52.10.20.30  │   │  │ ← Has Elastic IP
        │  │  └─────────────────────┘   │  │
        │  │                            │  │
        │  │  Route Table (public-rt):  │  │
        │  │   0.0.0.0/0 → igw-xxx ✅  │  │ ← NAT uses this to reach internet
        │  └────────────────────────────┘  │
        │                                   │
        │  ┌──── Private Subnet ─────────┐  │
        │  │  10.0.2.0/24               │  │
        │  │                            │  │
        │  │  [EC2: 10.0.2.5]  ─────────┼──┼──→ NAT GW → Internet
        │  │  [RDS: 10.0.2.20] ─────────┼──┼──→ NAT GW → Internet
        │  │                            │  │
        │  │  Route Table (private-rt): │  │
        │  │   0.0.0.0/0 → nat-xxx ✅  │  │ ← Routes through NAT
        │  └────────────────────────────┘  │
        └───────────────────────────────────┘
```

---

## 🔑 Key Facts About NAT Gateway

| Property | Detail |
|----------|--------|
| **Subnet placement** | MUST be in a **PUBLIC** subnet |
| **Elastic IP** | Requires one Elastic IP address |
| **Direction** | **Outbound only** — no inbound from internet |
| **Managed** | Fully managed by AWS — no patching/maintenance |
| **Bandwidth** | Scales from 5 Gbps → 100 Gbps automatically |
| **AZ-specific** | One NAT per AZ — NOT redundant across AZs |
| **Protocols** | TCP, UDP, ICMP |
| **Cost** | ~$0.045/hour + $0.045/GB processed (~$32/month base) |

---

## 🔄 How NAT Gateway Works — Packet Flow

### Private EC2 → Internet
```
Step 1: Private EC2 sends packet
        Source IP:      10.0.2.5   (private IP — not routable on internet)
        Destination IP: 91.189.91.42 (ubuntu apt server)

Step 2: Private subnet route table
        Destination 91.x.x.x → matches 0.0.0.0/0 → NAT Gateway

Step 3: NAT Gateway translates source IP
        Replaces: 10.0.2.5
        With:     52.10.20.30 (NAT's Elastic IP)
        Tracks:   This connection in NAT table

Step 4: Packet goes via public subnet route
        0.0.0.0/0 → IGW → Internet

Step 5: Server responds to 52.10.20.30
        NAT Gateway receives it
        Checks NAT table → maps back to 10.0.2.5
        Forwards to private EC2 ✅

Internet never knows the real IP is 10.0.2.5!
```

### Internet tries to initiate connection to private EC2
```
Internet sends: Destination = 52.10.20.30 (NAT's EIP)
NAT Gateway checks: Is there an established session for this?
  → NO (internet can't initiate)
  → DROPS the packet ✅

Private EC2 is protected!
```

---

## 🛠️ Hands-On: Create NAT Gateway Step by Step

### Step 1: Allocate Elastic IP (NAT needs it)
```
VPC → Elastic IPs → Allocate Elastic IP Address
  → Network border group: ap-south-1
  → Allocate

Note your Elastic IP (e.g., 52.30.40.50)
```

### Step 2: Create NAT Gateway
```
VPC → NAT Gateways → Create NAT Gateway

  Name:              my-nat-gateway
  Subnet:            public-subnet-1a   ⚠️ MUST BE PUBLIC!
  Connectivity type: Public
  Elastic IP:        Select 52.30.40.50 (or click Allocate)

  → Create NAT Gateway

Status: Pending → Available (takes ~1-2 minutes)
```

### Step 3: Update Private Subnet's Route Table
```
VPC → Route Tables → Select: private-rt

Routes tab → Edit Routes → Add Route:
  Destination: 0.0.0.0/0
  Target:      NAT Gateway → nat-xxxxxxxx

→ Save Changes

Final private-rt routes:
  ┌──────────────────┬─────────────┐
  │ 10.0.0.0/16     │ local       │
  │ 0.0.0.0/0       │ nat-xxxxxxx │ ← Added now
  └──────────────────┴─────────────┘
```

### Step 4: Test from Private EC2

```bash
# Connect to private EC2 via Bastion Host or SSM Session Manager
# (private EC2 has no public IP)

# Test outbound internet (should work now)
ping 8.8.8.8
# PING 8.8.8.8 ... 64 bytes from 8.8.8.8 ✅

curl -I https://google.com
# HTTP/2 200 ✅

# Download packages
sudo apt update
# Get:1 http://ap-south-1.ec2.archive.ubuntu.com/ubuntu ... ✅

# Check what IP internet sees (should be NAT's Elastic IP)
curl ifconfig.me
# 52.30.40.50 ← NAT Gateway's Elastic IP ✅
```

---

## 🏗️ High Availability NAT Gateway Setup

### Problem with Single NAT Gateway
```
Single NAT Gateway (bad for production):

  AZ-1 public subnet:  NAT GW 1
  AZ-1 private subnet: Route → NAT GW 1 ✅
  AZ-2 private subnet: Route → NAT GW 1 ← CROSS-AZ!

If AZ-1 fails → AZ-2 private instances lose internet!
Also: Cross-AZ data transfer costs extra
```

### ✅ Recommended: One NAT per AZ
```
AZ-1:
  public-subnet-1a:  NAT Gateway 1 (EIP: 52.x.x.1)
  private-subnet-1a: Route → NAT GW 1 (same AZ)

AZ-2:
  public-subnet-1b:  NAT Gateway 2 (EIP: 52.x.x.2)
  private-subnet-1b: Route → NAT GW 2 (same AZ)

Benefit: If AZ-1 fails → AZ-2 unaffected ✅
         No cross-AZ data transfer charges ✅
Cost:    2x NAT Gateway price (~$64/month)
```

### Route Table Setup for Multi-AZ HA
```
private-rt-az1 (for AZ-1 private subnet):
  0.0.0.0/0 → nat-gw-1  ← NAT in same AZ

private-rt-az2 (for AZ-2 private subnet):
  0.0.0.0/0 → nat-gw-2  ← NAT in same AZ
```

---

## 💰 NAT Gateway Pricing

```
Region: ap-south-1 (Mumbai)

Charges:
  Hourly:          $0.045/hour per NAT Gateway
  Monthly (1 NAT): $0.045 × 24 × 30 = $32.40/month
  Data processed:  $0.045 per GB

Example cost:
  2 NAT Gateways (HA) + 100 GB/month data:
  = (2 × $32.40) + (100 × $0.045)
  = $64.80 + $4.50
  = $69.30/month

Cost-saving tip:
  Use VPC Endpoints for S3 and DynamoDB → FREE, bypasses NAT!
  Can save significant $ if you do a lot of S3 traffic.
```

---

## 🆚 NAT Gateway vs NAT Instance

| Feature | NAT Gateway ✅ | NAT Instance ⚠️ |
|---------|--------------|----------------|
| **Management** | Fully managed (AWS) | You manage EC2 |
| **Availability** | Highly available in AZ | Single EC2 — SPOF |
| **Bandwidth** | 5–100 Gbps auto-scale | Limited by instance type |
| **Cost** | ~$32/month | Cheaper (t3.nano ~$3/month) |
| **Security Groups** | Cannot apply | Can apply SG |
| **Bastion Host** | No | Can double as bastion |
| **Port Forwarding** | Not supported | Supported |
| **Source/Dest Check** | N/A | Must DISABLE on instance |
| **Recommended** | ✅ Production | Legacy / Learning only |

### NAT Instance Setup (Legacy — for reference only)
```bash
# On the NAT Instance EC2:
# 1. Must be in PUBLIC subnet
# 2. Disable Source/Destination Check:
#    EC2 → Instance → Actions → Networking → Change Source/Dest Check → Disable

# 3. Enable IP forwarding on the OS:
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

# 4. Add iptables NAT rule:
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

## 🔢 NAT Gateway Types

| Type | EIP | Route Through | Use Case |
|------|-----|--------------|----------|
| **Public NAT** | Yes (required) | → IGW → Internet | Private subnet → public internet |
| **Private NAT** | No | → Transit GW | Private subnet → other VPCs/on-prem |

### Private NAT Gateway (Advanced)
```
Use case: Connect private subnets across VPCs without internet
  VPC-A Private Subnet → Private NAT → Transit Gateway → VPC-B

VPC → NAT Gateways → Create
  Connectivity Type: Private  ← No EIP needed
```

---

## 💡 Cost Optimization Tips

### Tip 1: Use VPC Endpoints for S3 & DynamoDB
```
Without VPC Endpoint:
  EC2 → NAT Gateway ($) → Internet → S3

With VPC Gateway Endpoint (FREE!):
  EC2 → VPC Endpoint (free) → S3

Setup:
VPC → Endpoints → Create Endpoint
  Service: com.amazonaws.ap-south-1.s3
  Type: Gateway
  Route Table: private-rt
  → Create

Route automatically added:
  pl-xxxxx (S3 prefix list) → vpce-xxx

Now S3 traffic BYPASSES NAT Gateway → saves money!
```

### Tip 2: Use NAT Gateway Schedules for Dev/Test
```bash
# Stop NAT Gateway at night (dev/test only!)
# Note: Can't "stop" NAT GW — must delete and recreate

# Better: Use Lambda + EventBridge to schedule
# Or: Use single NAT for dev (not HA)
```

### Tip 3: CloudWatch to Monitor NAT Usage
```
CloudWatch → Metrics → VPC → NAT Gateway Metrics:
  BytesInFromDestination    ← Data downloaded from internet
  BytesOutToDestination     ← Data uploaded to internet
  ConnectionAttemptCount    ← Connection attempts
  ErrorPortAllocation       ← Port exhaustion (scale issue)
```

---

## 📊 NAT Gateway CloudWatch Metrics

| Metric | Description | Alert When |
|--------|-------------|-----------|
| BytesInFromDestination | Data received from internet | Unusually high |
| BytesOutToDestination | Data sent to internet | Unusually high |
| ConnectionAttemptCount | Outbound connection attempts | Spike |
| ConnectionEstablishedCount | Successful connections | Drop |
| ErrorPortAllocation | Failed port allocation | > 0 |
| PacketsDropCount | Packets dropped | > 0 |

---

## ⚠️ Common Mistakes

### Mistake 1: NAT Gateway in PRIVATE subnet
```
❌ WRONG:
  Private subnet → Route → NAT GW (in private subnet)
  NAT Gateway has no internet → Can't forward traffic!

✅ CORRECT:
  Private subnet → Route → NAT GW (in PUBLIC subnet)
  Public subnet has route to IGW → NAT can reach internet
```

### Mistake 2: Public subnet has no IGW route
```
❌ WRONG:
  public-rt: only has local route
  NAT Gateway is in public subnet but public subnet can't reach internet!

✅ CORRECT:
  public-rt must have: 0.0.0.0/0 → igw-xxx
```

### Mistake 3: Private subnet route points to wrong NAT
```
❌ WRONG (cross-AZ, costly):
  AZ-2 private subnet → NAT GW in AZ-1

✅ CORRECT (same AZ, cheaper):
  AZ-2 private subnet → NAT GW in AZ-2
```

### Mistake 4: Forgot to allocate Elastic IP
```
Error: "Elastic IP address is required for a public NAT gateway"
Fix: VPC → Elastic IPs → Allocate → then use in NAT creation
```

### Mistake 5: Leaving NAT Gateway running when not needed
```
NAT Gateway charges by the hour even when idle!
Tip: Delete NAT GW when not using (dev/test env)
     Recreate when needed (takes only 1-2 minutes)
```

---

## 📋 NAT Gateway Checklist

```
Creating NAT Gateway:
  □ Elastic IP allocated?
  □ Subnet selected is PUBLIC (not private)?
  □ Status = Available (not Pending)?

After Creation:
  □ Private subnet's route table updated?
  □ Route: 0.0.0.0/0 → nat-xxxxxxxx added?
  □ Private subnet associated with private route table?

Testing:
  □ Connected to private EC2 via Bastion/SSM?
  □ ping 8.8.8.8 works from private EC2?
  □ apt update / yum update works?
  □ curl ifconfig.me shows NAT's EIP?
```

---

## ✅ Advantages

- **Fully managed** — no OS patching, no configuration
- **Auto-scales** — from 5 Gbps to 100 Gbps automatically
- **Highly available** within AZ
- **Outbound-only protection** — internet can't reach private servers
- **Supports TCP, UDP, ICMP**
- **Static EIP** — your private servers always use same outbound IP

## ❌ Disadvantages

- **Expensive** — ~$32/month + data processing charges
- **Not free tier** eligible
- **AZ-specific** — need one per AZ for HA (doubles cost)
- **No SG** — can't apply Security Groups to NAT Gateway
- **No inbound** — cannot use for inbound public traffic

---

## 🏆 Best Practices

- Always place NAT Gateway in **PUBLIC subnet**
- Deploy **one NAT per AZ** for production HA
- Use **VPC Gateway Endpoints** for S3/DynamoDB (free, no NAT needed)
- **Monitor** BytesOutToDestination — unexpected spike = data exfiltration risk
- **Delete** NAT Gateways in dev/test when not in use (saves money)
- **Release Elastic IP** after deleting NAT GW (unused EIPs also cost money)
- Tag clearly: `Name: nat-gw-ap-south-1a`, `Env: prod`

---

## 📚 Related Files

- [03_VPC.md](03_VPC.md) — VPC fundamentals
- [19_Internet_Gateway.md](19_Internet_Gateway.md) — IGW (needed by NAT)
- [20_Route_Tables.md](20_Route_Tables.md) — Route table configuration
- [13_Peering_Connection.md](13_Peering_Connection.md) — Alternative for VPC-to-VPC
