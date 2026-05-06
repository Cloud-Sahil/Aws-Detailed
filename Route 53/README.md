# 🌐 Route 53 - DNS & Traffic Management

## 📌 What is Route 53?

Amazon Route 53 is a highly available and scalable **Domain Name System (DNS) web service**. It routes end users to internet applications by translating domain names into IP addresses.

> Route 53 = DNS + Health Checks + Traffic Routing — the "phone book" of the internet for your AWS services.

---

## 🏗️ Route 53 Architecture

```
User types: www.myapp.com
      ↓
Route 53 Resolver
      ↓
Hosted Zone (myapp.com)
      ↓
DNS Records
      ├── A Record: myapp.com → 52.x.x.x (EC2 IP)
      ├── CNAME: www → myapp.com
      ├── ALIAS: myapp.com → alb-dns.amazonaws.com
      └── MX: myapp.com → mail.myapp.com
```

---

## 📋 DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Domain → IPv4 | myapp.com → 52.10.20.30 |
| **AAAA** | Domain → IPv6 | myapp.com → 2001:db8::1 |
| **CNAME** | Domain → Domain | www → myapp.com |
| **ALIAS** | Domain → AWS resource | myapp.com → alb-xxx.amazonaws.com |
| **MX** | Mail server | myapp.com → mail.myapp.com |
| **TXT** | Text records | SPF, DKIM, verification |
| **NS** | Name servers | Delegates zone to Route 53 |
| **SOA** | Zone authority | Auto-created |

> ✅ Use **ALIAS** (not CNAME) for root domain pointing to AWS resources — it's free and faster!

---

## 🔀 Routing Policies

| Policy | Description | Use Case |
|--------|-------------|----------|
| **Simple** | Single record → one IP | Basic websites |
| **Weighted** | Split traffic by % | A/B testing, gradual migration |
| **Latency** | Route to lowest latency region | Global apps |
| **Failover** | Primary/Secondary setup | DR scenarios |
| **Geolocation** | Route by user's country | Compliance, localization |
| **Geoproximity** | Route by geographic bias | Fine-grained geographic routing |
| **Multivalue** | Multiple IPs, health checked | Load distribution |
| **IP-Based** | Route by client IP | VPN traffic routing |

---

## 🛠️ Hands-On: Traffic Policy with Weighted Routing

```
Route 53 → Traffic Policies → Create Traffic Policy
  → Name: my-weighted-policy
  → DNS record type: A

Add routing rules:
  → Weighted → Weight 80 → Endpoint: 192.168.0.0 (main server)
  → Weighted → Weight 20 → Endpoint: 192.178.0.0 (test server)

Create Policy Records:
  → Hosted Zone: your hosted zone
  → DNS Name: myapp.com
  → TTL: 60
  → Create Policy Records
```

### Result: 80% traffic to main, 20% to test server

---

## 🏥 Health Checks

```
Route 53 → Health Checks → Create
  → Protocol: HTTP
  → IP/Domain: your-server-ip
  → Port: 80
  → Path: /health
  → Interval: 30 seconds
  → Failure threshold: 3
  → Create

→ Attach to Routing Policy for automatic failover
```

---

## ✅ Advantages

- **Highly available** — 100% SLA
- **Global** — DNS servers worldwide
- **Multiple routing policies** for traffic management
- **Health checks** built-in
- **Integrates** with all AWS services (ALB, CloudFront, S3)

## ❌ Disadvantages

- DNS propagation can take time (TTL dependent)
- Cost per hosted zone ($0.50/month)
- Learning curve for traffic policies

## 🏆 Best Practices

- Use **low TTL** (60s) during migration, high TTL (300s) for stability
- Use **ALIAS records** for AWS resources (free, faster)
- Enable **health checks** for failover routing
- Use **private hosted zones** for internal DNS (within VPC)
