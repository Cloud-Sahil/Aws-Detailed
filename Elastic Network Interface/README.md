# 🔌 ENI - Elastic Network Interface (NIC)

## 📌 What is ENI?

An Elastic Network Interface (ENI) is a **virtual network interface card (NIC)** that you can attach to an EC2 instance. It acts as a virtual network adapter providing networking capabilities to your instances.

> Think of ENI as a **virtual network card** — just like a physical NIC in a server, but in the cloud. You can add multiple NICs to one instance.

---

## 🏗️ ENI Architecture

```
EC2 Instance (ap-south-1a)
├── Primary ENI (eth0) - Auto created
│   ├── Primary Private IP: 10.0.0.5
│   ├── Secondary Private IP: 10.0.0.6
│   ├── Elastic IP: 52.x.x.x
│   ├── MAC Address: 0e:1d:2c:3b:4a:5f
│   └── Security Groups: [web-sg]
│
└── Secondary ENI (eth1) - Manually attached
    ├── Private IP: 10.0.1.10
    ├── Elastic IP: 54.x.x.x
    ├── Security Groups: [admin-sg]
    └── Same AZ as EC2!
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **Primary ENI** | Auto-created, can't be detached |
| **Secondary ENI** | Manually attached, can be moved |
| **Private IP** | Internal VPC IP (persists on ENI) |
| **Elastic IP** | Static public IP associated with ENI |
| **MAC Address** | Fixed per ENI (useful for licensing) |
| **Security Group** | Attached at ENI level |
| **Source/Dest Check** | Must disable for NAT instances |

---

## 🛠️ Hands-On: Create & Attach ENI

### Step 1: Create EC2 Instance
```
EC2 → Launch Instance → Ubuntu 22.04 → t2.micro → Launch
Note the Availability Zone (must match ENI!)
```

### Step 2: Create Network Interface
```
EC2 → Network Interfaces → Create Network Interface
  → Description: my-secondary-nic
  → Subnet: Same subnet/AZ as your EC2!
  → Security Group: your-security-group
  → Create Network Interface
```

### Step 3: Attach ENI to EC2
```
Network Interfaces → Select ENI → Actions → Attach
  → Instance: Select your EC2 instance
  → Device Index: 1 (eth1)
  → Attach
```

### Step 4: Associate Elastic IP
```
EC2 → Elastic IPs → Allocate Elastic IP → Allocate
  → Actions → Associate Elastic IP Address
  → Resource Type: Network Interface
  → Network Interface: Select your ENI
  → Private IP: Select the private IP
  → Associate
```

### Step 5: Verify in EC2
```bash
# Connect to EC2
ssh -i key.pem ubuntu@<primary-public-ip>
sudo -i

# See network interfaces
ip addr show
# Should show eth0 (primary) and eth1 (secondary)

ifconfig
# Shows both interfaces
```

---

## 🔄 ENI Use Cases

| Use Case | Description |
|----------|-------------|
| **Dual-homed** | Instance in two subnets simultaneously |
| **Network appliance** | Firewall, router with multiple interfaces |
| **Management NIC** | Separate management network |
| **License by MAC** | Software licensed to specific MAC address |
| **Failover** | Move ENI (with IP) to another instance instantly |
| **Low-budget HA** | Move ENI to standby instead of ELB |

---

## 🔁 ENI Failover Pattern

```
Primary EC2 (running) → ENI with Elastic IP 52.x.x.x
         ↓ (instance fails)
Detach ENI → Attach to Standby EC2
         ↓
Same IP 52.x.x.x now points to Standby EC2
Failover time: ~1 minute
```

---

## ✅ Advantages

- Multiple IPs on one instance
- Multiple security groups
- Fixed MAC address for licensing
- Move ENI between instances (IP follows ENI)
- Free to create

## ❌ Disadvantages

- Must be in same AZ as EC2
- Limited ENIs per instance type (e.g., t2.micro = 2 ENIs)
- Primary ENI cannot be detached

## 🏆 Best Practices

- Use ENIs for **network appliances** requiring multiple interfaces
- Always create ENI in **same AZ** as target instance
- Use **Elastic IP on ENI** for static IPs that can move
- **Name your ENIs** clearly for management
