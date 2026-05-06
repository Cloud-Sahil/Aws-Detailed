# 📈 Auto Scaling - EC2 Auto Scaling Groups

## 📌 What is Auto Scaling?

AWS Auto Scaling automatically **adjusts the number of EC2 instances** based on demand. It ensures you have the right number of instances available to handle the load.

> Think of Auto Scaling as a **smart manager** who adds more staff when it gets busy and reduces staff during slow hours — automatically.

---

## 🏗️ Auto Scaling Architecture

```
CloudWatch Alarm (CPU > 65%)
        ↓
Auto Scaling Group
        ↓ (scales out)
    [EC2-1] [EC2-2] [EC2-3] [EC2-4] [EC2-5]
        ↑
Application Load Balancer
        ↑
     Internet
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **Launch Template** | Blueprint for EC2 instances (AMI, type, SG, user data) |
| **Launch Configuration** | Older version of Launch Template (deprecated) |
| **ASG (Auto Scaling Group)** | Group of EC2s managed together |
| **Desired Capacity** | Current number of instances wanted |
| **Minimum Capacity** | Minimum instances (floor) |
| **Maximum Capacity** | Maximum instances (ceiling) |
| **Scaling Policy** | Rules for when to add/remove instances |
| **Cooldown Period** | Wait time between scaling actions |

---

## 📐 Scaling Policies

| Policy Type | Description | Use Case |
|-------------|-------------|----------|
| **Target Tracking** | Maintain metric at target value | CPU at 65% |
| **Step Scaling** | Scale in steps based on alarm | CPU > 70% → add 2 |
| **Simple Scaling** | One action per alarm | CPU > 80% → add 1 |
| **Scheduled Scaling** | Scale at specific time | 9 AM add 5, 6 PM remove 3 |
| **Predictive Scaling** | ML-based future scaling | Traffic patterns |

---

## 🛠️ Hands-On: Create Auto Scaling Group

### Step 1: Create Launch Template
```
EC2 → Launch Templates → Create Launch Template
  → Name: nginx-lt
  → Template version description: v1
  → AMI: Ubuntu 22.04 LTS (Quick Start)
  → Instance Type: t2.micro
  → Key Pair: your-key
  → Security Groups: your-sg (port 80, 22)
  → Advanced Details → User Data:

#!/bin/bash
apt update -y
apt install nginx -y
systemctl start nginx
systemctl enable nginx

  → Create Launch Template
```

### Step 2: Create Auto Scaling Group
```
EC2 → Auto Scaling Groups → Create
```

#### Page 1: Choose launch template
```
Name: my-asg
Launch Template: nginx-lt (v1)
→ Next
```

#### Page 2: Choose instance launch options
```
VPC: default or your VPC
Availability Zones: Select ALL (ap-south-1a, 1b, 1c)
→ Next
```

#### Page 3: Configure advanced options
```
Load Balancing: Attach to ALB (optional)
Health Checks: EC2 health checks
→ Next
```

#### Page 4: Configure group size and scaling
```
Desired capacity: 3
Minimum capacity: 2
Maximum capacity: 6

Automatic Scaling:
  → Target tracking scaling policy
  → Metric type: Average CPU Utilization
  → Target value: 65
  → Instance warmup: 120 seconds

→ Next
```

#### Page 5: Notifications (optional)
```
→ Next
```

#### Page 6: Tags
```
Key: Name
Value: nginx-asg-instance
→ Next
```

#### Page 7: Review & Create
```
→ Create Auto Scaling Group
```

### Step 3: Verify Instances
```
EC2 → Instances → Filter by tag Name: nginx-asg-instance
  → Should see 3 instances launching
```

---

## 🔥 Load Testing with Stress

```bash
# Connect to an ASG instance
ssh -i key.pem ubuntu@<public-ip>
sudo apt install stress -y

# Stress test (generates CPU load)
stress --cpu 1000 --io 4 --vm 2 --vm-bytes 128M --timeout 300s

# Monitor in CloudWatch / ASG Activity History
# CPU goes above 65% → ASG scales out!
```

---

## ⚡ Scaling Scenarios

### Scale Out (Add instances)
```
Trigger: CPU > 65% for 5 minutes
Action: Add 1 instance
Cooldown: 300 seconds (don't scale again for 5 mins)
```

### Scale In (Remove instances)
```
Trigger: CPU < 30% for 10 minutes
Action: Remove 1 instance
Termination Policy: Oldest instance first (default)
```

---

## 🏷️ Termination Policies

| Policy | Description |
|--------|-------------|
| **Default** | Balance AZs, then oldest launch config |
| **OldestInstance** | Terminate oldest instance first |
| **NewestInstance** | Terminate newest first |
| **OldestLaunchTemplate** | Terminate instances with oldest template |
| **ClosestToNextInstanceHour** | Minimize billing (spot) |

---

## 📊 Instance Refresh

Update all instances when launch template changes:

```
ASG → Instance Refresh → Start
  → Minimum healthy percentage: 90%
  → Warm up: 120 seconds
  → Start

Replaces instances one by one with new config
No downtime!
```

---

## 🔄 ASG Lifecycle Hooks

Pause instances during launch/terminate for custom actions:

```
Launching:
  [Pending] → [Pending:Wait] → (run scripts) → [Pending:Proceed] → [InService]

Terminating:
  [Terminating] → [Terminating:Wait] → (drain/backup) → [Terminating:Proceed] → [Terminated]
```

---

## 📈 Scheduled Scaling

```python
# Scale up before business hours
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-up-morning \
  --recurrence "0 9 * * MON-FRI" \
  --desired-capacity 5

# Scale down after hours
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-asg \
  --scheduled-action-name scale-down-evening \
  --recurrence "0 18 * * MON-FRI" \
  --desired-capacity 2
```

---

## 🔧 Update User Data After ASG Launch

```
EC2 → Instance → Actions → Instance Settings → Edit User Data
  (Must stop instance first)
  → Add/modify user data script
  → Start instance
```

---

## 🔄 ASG + ALB Integration

```
1. Create ASG with launch template
2. Create Target Group
3. ASG → Edit → Load Balancing → Add ALB Target Group

Now: New instances auto-register with ALB!
     Terminated instances auto-deregister!
```

---

## ✅ Advantages

- **Automatic scaling** — no manual intervention
- **Cost optimization** — pay only for what you need
- **High availability** — spread across multiple AZs
- **Health replacement** — auto-replace unhealthy instances
- **Predictive** — ML-based proactive scaling
- **Free** — no charge for ASG, only for instances

---

## ❌ Disadvantages

- **Launch time** (~1-2 minutes to boot new EC2)
- **Stateful apps** need careful design (sessions)
- **Cooldown periods** can delay response to sudden spikes
- **Configuration complexity** for fine-tuning

---

## 🏆 Best Practices

- Use **Launch Templates** (not Launch Configurations — deprecated)
- Set **appropriate cooldown** (too short = thrashing)
- Use **Target Tracking** for most use cases (simplest)
- Spread instances across **all AZs**
- Use **Instance Warmup** to not count new instance metrics immediately
- Monitor with **CloudWatch ASG metrics**
- Use **Mixed Instances Policy** with Spot + On-Demand for cost savings
- Enable **scale-in protection** for critical instances

---

## 🔢 ASG Metrics in CloudWatch

| Metric | Description |
|--------|-------------|
| GroupDesiredCapacity | Target number of instances |
| GroupInServiceInstances | Running instances |
| GroupPendingInstances | Launching instances |
| GroupTerminatingInstances | Terminating instances |
| GroupTotalInstances | All instances |

---

## 📚 Related Services

- **EC2** — Instances managed by ASG
- **ALB/NLB** — Distribute traffic across ASG
- **CloudWatch** — Trigger scaling based on metrics
- **SNS** — Notifications on scaling events
- **ELB** — Health checks for ASG instances
