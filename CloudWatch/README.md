# 📊 CloudWatch - Monitoring & Observability

## 📌 What is CloudWatch?

Amazon CloudWatch is a **monitoring and observability service** that collects metrics, logs, and events from AWS resources and applications. It provides actionable insights to monitor applications, respond to performance changes, and optimize resource utilization.

> Think of CloudWatch as your **AWS dashboard + alarm system** — like a control room that shows health, performance, and alerts you when something goes wrong.

---

## 🏗️ CloudWatch Architecture

```
AWS Services / Applications
    ↓ (metrics, logs, events)
CloudWatch
    ├── Metrics        ← Numerical time-series data (CPU, memory)
    ├── Logs           ← Text log data (application, access logs)
    ├── Alarms         ← Trigger actions when metrics cross threshold
    ├── Dashboards     ← Visual graphs and panels
    ├── Events/Rules   ← React to state changes
    └── Insights       ← Log analytics with query language
```

---

## 📋 CloudWatch Components

| Component | Description |
|-----------|-------------|
| **Metrics** | Time-series data points (CPUUtilization, NetworkIn) |
| **Namespaces** | Container for metrics (AWS/EC2, AWS/S3) |
| **Dimensions** | Filters for metrics (InstanceId, AutoScalingGroupName) |
| **Statistics** | Aggregations (Average, Sum, Min, Max, p99) |
| **Alarms** | Trigger actions based on metric thresholds |
| **Logs** | Collect and store log data |
| **Log Groups** | Container for log streams |
| **Log Streams** | Sequence of logs from one source |
| **Dashboards** | Custom visual monitoring panels |

---

## 📈 Default EC2 Metrics (Free)

| Metric | Description |
|--------|-------------|
| CPUUtilization | % CPU in use |
| NetworkIn/Out | Network bytes in/out |
| DiskReadOps/WriteOps | Disk operations |
| StatusCheckFailed | Instance/system check failures |

> ⚠️ **RAM/Memory is NOT included by default** — requires CloudWatch Agent

## 📈 Detailed Monitoring Metrics (Paid)

- 1-minute granularity (default is 5 minutes)
- Enable per instance: EC2 → Monitoring → Enable Detailed Monitoring

---

## 🛠️ Hands-On: Create CloudWatch Dashboard

### Step 1: Create EC2 Instance
```
EC2 → Launch Instance (t2.micro, Ubuntu) → Launch
```

### Step 2: Create Dashboard
```
CloudWatch → Dashboards → Create Dashboard
  → Name: my-ec2-dashboard
  → Create Dashboard

Add Widget:
  → Data source: CloudWatch
  → Widget type: Line (metrics)
  → Next
  → Metrics → EC2 → Per-Instance Metrics
  → Search/paste your Instance ID
  → Select: CPUUtilization
  → Create Widget
  → Save Dashboard
```

---

## 🚨 Hands-On: Create CloudWatch Alarm

### Alarm on CPU Utilization
```
CloudWatch → Alarms → Create Alarm
  → Select Metric → EC2 → Per-Instance Metrics
  → Paste EC2 Instance ID → Select: CPUUtilization
  → Select Metric

Specify metric & conditions:
  → Statistic: Average
  → Period: 5 minutes
  → Threshold type: Static
  → Whenever CPUUtilization is: Greater than
  → Than: 80 (trigger when CPU > 80%)
  → Next

Configure actions:
  → Notification: Remove (or add SNS topic for email)
  → EC2 Action → In Alarm → Stop/Reboot/Terminate/Recover instance
  → Next

Add name:
  → Alarm Name: high-cpu-alarm
  → Next → Create Alarm
```

### Trigger the Alarm (Stress Test)
```bash
# Connect to EC2
sudo apt update
sudo apt install stress -y
stress --help

# Run stress test (causes alarm to trigger)
stress --cpu 100 --io 4 --vm 2 --vm-bytes 128M --timeout 1000s

# Watch CloudWatch alarm change state:
# OK → INSUFFICIENT_DATA → ALARM
```

---

## 📝 CloudWatch Logs

### CloudWatch Agent Setup (for Memory, Disk, App Logs)

#### Step 1: Create EC2 with Amazon Linux
```
EC2 → Launch Instance
  → Quick Start: Amazon Linux 2023
  → t2.medium
  → Launch → Connect
```

#### Step 2: Create IAM Role
```
IAM → Roles → Create Role
  → AWS Service → EC2
  → Add Policy: CloudWatchAgentServerPolicy
  → Name: ec2-cw-agent-role
  → Create Role
```

#### Step 3: Attach Role to EC2
```
EC2 → Instance → Actions → Security → Modify IAM Role
  → Select: ec2-cw-agent-role → Update
```

#### Step 4: Install CloudWatch Agent
```bash
sudo -i
sudo yum update -y

# Download agent
wget https://amazoncloudwatch-agent.s3.amazonaws.com/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm

# Install
sudo rpm -U ./amazon-cloudwatch-agent.rpm

# Configure (wizard)
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

# Wizard answers (for basic setup):
# 1 (Linux) → 1 (EC2) → 2 (no StatsD) → 2 (no CollectD)
# 2 (default metrics) → 1 (yes, monitor CPU) → 1 (yes interval)
# 1 (yes memory) → 1 (yes disk) → 2 (no net) → 1 (yes processes)
# 2 (no) → 1 (yes logs) → 1 (yes) → /var/log/nginx/access.log
# → access.log → (enter) → 2 (done)

# Start agent
sudo amazon-cloudwatch-agent-ctl -a start \
  -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s

# Check status
systemctl status amazon-cloudwatch-agent
```

#### Step 5: View Logs in CloudWatch
```
CloudWatch → Logs → Log Groups
  → Look for: /var/log/nginx/access.log
  → Log Streams → View events
```

---

## 📊 CloudWatch Metrics — Custom Metrics

```bash
# Push custom metric from EC2
aws cloudwatch put-metric-data \
  --metric-name MyCustomMetric \
  --namespace MyApp \
  --value 42 \
  --unit Count \
  --region ap-south-1
```

---

## 🔍 CloudWatch Log Insights

Query logs using CloudWatch Logs Insights:

```sql
-- Find top 10 error logs
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 10

-- Count HTTP status codes
fields @timestamp, @message
| parse @message "HTTP/1.1\" * " as status
| stats count(*) as count by status
| sort count desc
```

---

## ⏰ CloudWatch Events / EventBridge

React to AWS events:

```
Example: Auto-stop EC2 instances at 10 PM daily
EventBridge → Rules → Create Rule
  → Schedule: cron(0 22 * * ? *)
  → Target: EC2 → StopInstances
  → Target EC2 IDs: i-1234567890abcdef0
```

---

## 📊 Important CloudWatch Metrics by Service

### EC2
```
CPUUtilization, NetworkIn, NetworkOut, DiskReadBytes, StatusCheckFailed
```

### ELB/ALB
```
RequestCount, TargetResponseTime, HTTPCode_Target_4XX, HealthyHostCount
```

### RDS
```
CPUUtilization, DatabaseConnections, FreeableMemory, ReadLatency
```

### S3
```
BucketSizeBytes, NumberOfObjects, AllRequests
```

---

## ✅ Advantages

- **Unified monitoring** for all AWS services
- **Real-time** metrics and logs
- **Automated actions** via alarms
- **Cost optimization** insights
- **No infrastructure** to manage
- **Integration** with SNS, Lambda, Auto Scaling

---

## ❌ Disadvantages

- Default 5-minute metrics (1-minute costs extra)
- Log storage costs money
- Complex query syntax for Logs Insights
- Limited retention by default (need to set custom retention)

---

## 🏆 Best Practices

- Install **CloudWatch Agent** for memory/disk metrics
- Set up **alarms** for critical metrics (CPU, memory, disk)
- Use **Composite Alarms** to reduce alert noise
- Set **log retention** policies (not indefinite — costs money)
- Use **metric filters** to extract metrics from logs
- Create **dashboards** for each environment
- Use **anomaly detection** for automatic baselines

---

## 📚 Related Services

- **CloudTrail** — API call logging (who did what)
- **SNS** — Alert notifications from CloudWatch alarms
- **Lambda** — Triggered by CloudWatch events
- **Auto Scaling** — Scale based on CloudWatch metrics
- **EventBridge** — Newer version of CloudWatch Events
