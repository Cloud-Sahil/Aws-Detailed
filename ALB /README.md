# ⚖️ ALB - Application Load Balancer

## 📌 What is ALB?

AWS ALB (Application Load Balancer) is a **Layer 7 (HTTP/HTTPS) load balancer** that distributes incoming application traffic across multiple targets (EC2 instances, containers, IP addresses, Lambda) based on content of the request.

> Think of ALB as a **smart traffic cop** — it reads the URL, headers, and request content, then routes traffic to the right backend server.

---

## 🏗️ ALB Architecture

```
Internet
    ↓
ALB (Public, multi-AZ)
    ├── Listener: Port 80 (HTTP)
    │   └── Rules:
    │       ├── /cloth/* → Target Group: cloth-servers
    │       ├── /admin/* → Target Group: admin-servers
    │       └── Default  → Target Group: home-servers
    └── Listener: Port 443 (HTTPS)
        └── Certificate (ACM)

Target Groups:
  home-servers:   [EC2-1 (AZ-a)] [EC2-2 (AZ-b)]
  cloth-servers:  [EC2-3 (AZ-a)] [EC2-4 (AZ-b)]
  admin-servers:  [EC2-5 (AZ-a)]
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **Listener** | Port/protocol the ALB listens on (80, 443) |
| **Target Group** | Group of EC2s/IPs/Lambdas receiving traffic |
| **Rules** | Conditions for routing (path, host, headers) |
| **Health Check** | ALB checks if targets are healthy |
| **Sticky Sessions** | Route same user to same target |
| **SSL Termination** | HTTPS handled at ALB, HTTP to backend |

---

## 🔀 ALB Routing Rules

| Routing Type | Example | Use Case |
|--------------|---------|----------|
| **Path-based** | /api/* → api-servers | Microservices |
| **Host-based** | api.myapp.com → api-servers | Multi-tenant |
| **Header-based** | X-Version: 2 → v2-servers | A/B testing |
| **Method-based** | POST → write-servers | Read/write split |
| **Query string** | ?version=beta → beta-tg | Feature flags |
| **IP-based** | 10.0.0.0/8 → internal-tg | Internal routing |

---

## 🛠️ Hands-On: Create ALB with Two Backend Servers

### Step 1: Launch EC2 — Nginx Server
```
EC2 → Launch Instance
  Name: nginx-server
  AMI: Ubuntu 22.04
  Instance Type: t2.micro
  Security Group: Allow SSH(22), HTTP(80)
  Advanced Details → User Data:

#!/bin/bash
apt update -y
apt install nginx -y
echo "<h1>Hello from NGINX Server!</h1>" > /var/www/html/index.html
systemctl start nginx
systemctl enable nginx
```

### Step 2: Launch EC2 — Apache Server
```
EC2 → Launch Instance
  Name: apache-server
  AMI: Ubuntu 22.04
  Instance Type: t2.micro
  Advanced Details → User Data:

#!/bin/bash
apt update -y
apt install apache2 -y
echo "<h1>Hello from Apache Server!</h1>" > /var/www/html/index.html
systemctl start apache2
systemctl enable apache2
```

### Step 3: Verify Both Servers
```
Open new tab → paste nginx-public-ip → See nginx page
Open new tab → paste apache-public-ip → See apache page
```

### Step 4: Create Target Group
```
EC2 → Target Groups → Create Target Group
  → Target Type: Instances
  → Name: my-target-group
  → Protocol: HTTP
  → Port: 80
  → VPC: default
  → Health Check: HTTP, Path: /
  → Next
  → Register Targets: Select both nginx and apache EC2
  → Include as pending
  → Create Target Group
```

### Step 5: Create Application Load Balancer
```
EC2 → Load Balancers → Create Load Balancer
  → Application Load Balancer → Create
  → Name: my-alb
  → Scheme: Internet-facing
  → IP type: IPv4
  → VPC: default
  → Mappings: Select ALL Availability Zones
  → Security Groups: Select SG with port 80
  → Listeners: HTTP:80 → Forward to my-target-group
  → Create Load Balancer
```

### Step 6: Access via ALB DNS
```
Load Balancers → Select my-alb → DNS Name → Copy
Paste in browser → Traffic distributed between nginx & apache
Refresh multiple times to see different responses!
```

---

## 🎯 Mini Project: Path-Based Routing

### Scenario: E-commerce site with /cloth path
```
/ → home page (home target group)
/cloth/* → cloth page (cloth target group)
```

### Step 1: Launch Templates
```
Template 1: home
  User Data:
    #!/bin/bash
    apt update -y && apt install nginx -y
    echo "<h1>Welcome To My E-commerce Website</h1>" > /var/www/html/index.html
    systemctl start nginx && systemctl enable nginx

Template 2: cloth
  User Data:
    #!/bin/bash
    apt update -y && apt install nginx -y
    mkdir -p /var/www/html/cloth
    echo "<h1>CLOTH SALE!!! SALE!!!</h1>" > /var/www/html/cloth/index.html
    systemctl start nginx && systemctl enable nginx
```

### Step 2: Auto Scaling Groups
```
ASG-home:  launch template=home, desired=2, min=2, max=2
ASG-cloth: launch template=cloth, desired=2, min=2, max=2
```

### Step 3: Target Groups
```
TG-home:  register home instances, port 80
TG-cloth: register cloth instances, port 80
```

### Step 4: ALB Rules
```
ALB → Listeners → HTTP:80 → View/Edit Rules
  → Add Rule:
      Condition: Path Pattern = /cloth/*
      Action: Forward to TG-cloth
      Priority: 1
  → Add Rule:
      Condition: Default
      Action: Forward to TG-home

Test:
  http://alb-dns-name       → Home page
  http://alb-dns-name/cloth → Cloth page
```

---

## 🏥 Health Checks

```
Target Group → Health Check Settings:
  Protocol: HTTP
  Path: /health (or /)
  Port: traffic-port
  Healthy threshold: 2 (consecutive successes)
  Unhealthy threshold: 3 (consecutive failures)
  Timeout: 5 seconds
  Interval: 30 seconds

Status:
  healthy   → receives traffic
  unhealthy → removed from rotation
  draining  → in-flight requests finish, then removed
```

---

## 🔒 HTTPS with ACM Certificate

```
ACM → Request Certificate → Public Certificate
  → Domain: myapp.com, *.myapp.com
  → Validate via DNS → Create records in Route53

ALB → Listener → Add Listener → HTTPS:443
  → Default action: Forward to target group
  → Security Policy: ELBSecurityPolicy-TLS13-1-2-2021-06
  → Certificate: Select from ACM
  → Save
```

---

## 🔢 ALB vs NLB vs CLB

| Feature | ALB (Layer 7) | NLB (Layer 4) | CLB (Legacy) |
|---------|--------------|--------------|-------------|
| Protocol | HTTP, HTTPS, gRPC | TCP, UDP, TLS | HTTP, HTTPS, TCP |
| Routing | Content-based | IP/Port based | Basic |
| Performance | Good | Ultra-low latency | Poor |
| Static IP | Via NLB in front | Yes | No |
| WebSocket | Yes | Yes | Limited |
| Use Case | Web apps, microservices | Gaming, IoT, VoIP | Legacy only |

---

## 🔌 Port 8080 Host-Based Routing

```
Security Groups → Edit Inbound:
  Type: Custom TCP
  Port: 8080
  Source: 0.0.0.0/0
  Save

Access: http://alb-dns:8080
```

---

## ✅ Advantages

- **Content-based routing** — route by path, host, headers
- **Health checks** — automatic failover
- **SSL termination** — offload HTTPS from servers
- **Sticky sessions** — session persistence
- **CloudWatch integration** — detailed metrics
- **WAF integration** — protect from attacks
- **Auto scales** — handles traffic spikes

---

## ❌ Disadvantages

- **Layer 7 only** — not for raw TCP/UDP
- No static IP (use NLB for that)
- Slightly more latency than NLB

---

## 🏆 Best Practices

- Use **HTTPS (443)** with ACM certificates
- Enable **access logs** to S3 for debugging
- Set appropriate **health check intervals**
- Use **WAF** to protect from common attacks
- Enable **deletion protection** on production ALBs
- Use **target group deregistration delay** for graceful shutdowns
- Distribute targets across **multiple AZs**

---

## 📚 Related Services

- **Auto Scaling** — Scale EC2 behind ALB
- **ACM** — Free SSL certificates
- **WAF** — Web Application Firewall
- **Route 53** — DNS pointing to ALB
- **CloudWatch** — Monitor ALB metrics
