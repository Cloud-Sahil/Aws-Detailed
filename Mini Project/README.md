# 🏗️ Mini Project: EC2 + ALB + Auto Scaling (E-commerce Site)

## 📌 Project Overview

Build a **scalable e-commerce website** with:
- Two different pages: Home (`/`) and Cloth (`/cloth`)
- Auto Scaling Groups for each page
- Application Load Balancer for path-based routing
- High availability across multiple AZs

---

## 🏗️ Architecture

```
Internet
    ↓
Application Load Balancer (Public, multi-AZ)
    ├── Rule 1: /cloth/* → Target Group: cloth-tg
    │                           └── ASG: cloth-asg [EC2 cloth-1, EC2 cloth-2]
    │
    └── Default: /* → Target Group: home-tg
                          └── ASG: home-asg [EC2 home-1, EC2 home-2]

DNS: alb-xxxx.ap-south-1.elb.amazonaws.com
  /        → Home Page  (nginx serving /)
  /cloth   → Cloth Page (nginx serving /cloth/index.html)
```

---

## 🛠️ Step-by-Step Implementation

### Phase 1: Create Launch Templates

#### Launch Template 1: Home Page
```
EC2 → Launch Templates → Create Launch Template
  → Name: home-lt
  → Description: Home page template
  → AMI: Ubuntu 22.04 LTS (Quick Start)
  → Instance Type: t2.micro
  → Security Groups: your-sg (port 80, 22 allowed)

Advanced Details → User Data:
#!/bin/bash
apt update -y
apt install nginx -y
echo "<h1>Welcome To My E-commerce Website</h1>" > /var/www/html/index.html
systemctl start nginx
systemctl enable nginx

→ Create Launch Template
```

#### Launch Template 2: Cloth Page
```
EC2 → Launch Templates → Create Launch Template
  → Name: cloth-lt
  → Description: Cloth section template
  → AMI: Ubuntu 22.04 LTS (Quick Start)
  → Instance Type: t2.micro
  → Security Groups: your-sg

Advanced Details → User Data:
#!/bin/bash
apt update -y
apt install nginx -y
mkdir -p /var/www/html/cloth
echo "<h1>CLOTH SALE !!! SALE!!! SALE!!!</h1>" > /var/www/html/cloth/index.html
systemctl start nginx
systemctl enable nginx

→ Create Launch Template
```

---

### Phase 2: Create Auto Scaling Groups

#### ASG 1: Home
```
EC2 → Auto Scaling Groups → Create

Step 1: Name & Launch Template
  → Name: home-asg
  → Launch Template: home-lt
  → Next

Step 2: Network
  → VPC: default
  → AZs: Select ALL available
  → Next

Step 3: Load Balancing
  → Skip for now (will attach after creating ALB)
  → Next

Step 4: Capacity
  → Desired: 2
  → Minimum: 2
  → Maximum: 2
  → Next, Next

Step 6: Tags
  → Key: Name  Value: home-instance
  → Next → Create ASG
```

#### ASG 2: Cloth (same steps)
```
  → Name: cloth-asg
  → Launch Template: cloth-lt
  → Desired/Min/Max: 2/2/2
  → Tag: Name = cloth-instance
  → Create ASG
```

---

### Phase 3: Create Target Groups

#### Target Group 1: Home
```
EC2 → Target Groups → Create Target Group
  → Target Type: Instances
  → Name: home-tg
  → Protocol: HTTP | Port: 80
  → VPC: default
  → Health Check Path: /
  → Next

Register Targets:
  → Select home instances (from home-asg)
  → Add to Pending
  → Create Target Group
```

#### Target Group 2: Cloth
```
  → Name: cloth-tg
  → Protocol: HTTP | Port: 80
  → Health Check Path: /cloth/
  → Register cloth instances
  → Create Target Group
```

---

### Phase 4: Create Application Load Balancer

```
EC2 → Load Balancers → Create Load Balancer
  → Application Load Balancer → Create

Basic Config:
  → Name: ecommerce-alb
  → Scheme: Internet-facing
  → IP type: IPv4

Network:
  → VPC: default
  → Mappings: Select ALL Availability Zones

Security Groups:
  → Select SG with port 80 (and 443 if HTTPS)

Listeners and routing:
  → Protocol: HTTP | Port: 80
  → Default action: Forward to → home-tg

→ Create Load Balancer
```

---

### Phase 5: Add Path-Based Routing Rules

```
Load Balancers → Select ecommerce-alb
  → Listeners & Rules tab
  → Select HTTP:80 → Manage Rules

Add Rule:
  → Add Rule (blue button)
  → Name: cloth-rule
  
  Add Condition:
    → Path → /cloth/*
    → Confirm

  Add Action:
    → Forward to → cloth-tg

  Set Priority: 1 (lower = higher priority)
  → Next → Create

Now rules:
  Priority 1: /cloth/* → cloth-tg
  Default:    *        → home-tg
```

---

### Phase 6: Test the Application

```
Load Balancers → ecommerce-alb → DNS Name
Copy: ecommerce-alb-xxxx.ap-south-1.elb.amazonaws.com

Test 1: Home Page
  Browser: http://ecommerce-alb-xxxx.ap-south-1.elb.amazonaws.com/
  Expected: "Welcome To My E-commerce Website"

Test 2: Cloth Page
  Browser: http://ecommerce-alb-xxxx.ap-south-1.elb.amazonaws.com/cloth
  Expected: "CLOTH SALE !!! SALE!!! SALE!!!"
```

---

## 🔧 Enhancements to Add

### Add HTTPS (Production-ready)
```
1. Register domain in Route 53 (or use existing)
2. Request SSL certificate in ACM
3. ALB → Add Listener → HTTPS:443
4. Redirect HTTP → HTTPS
5. Route 53 → A Record (ALIAS) → ALB DNS
```

### Add Auto Scaling Policy
```
home-asg → Edit → Automatic Scaling:
  → Target Tracking: CPU = 65%
  → Min: 2, Max: 6
  → Warmup: 120s
```

### Add CloudWatch Dashboard
```
CloudWatch → Dashboard → Add widgets:
  → ALB RequestCount
  → ALB TargetResponseTime
  → ASG GroupInServiceInstances
  → EC2 CPUUtilization
```

---

## 🏆 What You Learn from This Project

| Skill | Service Used |
|-------|-------------|
| Infrastructure as template | Launch Templates |
| Automatic scaling | Auto Scaling Groups |
| Traffic distribution | Application Load Balancer |
| Path-based routing | ALB Rules |
| High availability | Multi-AZ deployment |
| Health monitoring | ALB Health Checks |

---

## 💡 Real-World Extension Ideas

1. **Add Database**: RDS MySQL in private subnet
2. **Add Cache**: ElastiCache Redis for sessions
3. **Add CDN**: CloudFront in front of ALB
4. **Add DNS**: Route 53 with custom domain
5. **Add Security**: WAF, ACM certificate, HTTPS
6. **Add CI/CD**: CodePipeline to auto-deploy updates
7. **Add Monitoring**: CloudWatch alarms + SNS notifications
