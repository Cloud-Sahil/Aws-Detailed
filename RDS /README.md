# 🗄️ RDS - Relational Database Service

## 📌 What is RDS?

Amazon RDS (Relational Database Service) is a **fully managed relational database service** that makes it easy to set up, operate, and scale SQL databases in the cloud. AWS handles hardware provisioning, database setup, patching, and backups.

> Think of RDS as **hiring a dedicated DBA team** — they handle all the operational heavy lifting (patching, backups, HA), and you just focus on your data and queries.

---

## 🏗️ RDS Architecture

```
AWS Region
└── VPC
    ├── Private Subnet AZ-1 (ap-south-1a)
    │   └── RDS Primary Instance
    │       ├── DB Engine (MySQL/Postgres/etc.)
    │       ├── EBS Storage (gp3/io1)
    │       └── Endpoint: mydb.xxx.ap-south-1.rds.amazonaws.com
    │
    └── Private Subnet AZ-2 (ap-south-1b)
        └── RDS Standby Instance (Multi-AZ)
            └── Synchronous replication from Primary

App Tier (EC2/Lambda) → RDS Endpoint → Primary (or Standby on failover)
```

---

## 📋 Supported Database Engines

| Engine | Versions | Use Case |
|--------|----------|----------|
| **MySQL** | 5.7, 8.0 | Web apps, WordPress |
| **PostgreSQL** | 13, 14, 15, 16 | Complex queries, GIS |
| **MariaDB** | 10.5, 10.6, 10.11 | MySQL compatible |
| **Oracle** | 19c, 21c | Enterprise apps |
| **SQL Server** | 2017, 2019, 2022 | .NET applications |
| **Aurora MySQL** | 3.x (MySQL 8.0 compat) | High performance |
| **Aurora PostgreSQL** | 15.x (PG compat) | High performance |

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **DB Instance** | The database server (compute + storage) |
| **DB Instance Class** | CPU/RAM tier (db.t3.micro, db.m5.large) |
| **Multi-AZ** | Automatic failover standby in another AZ |
| **Read Replica** | Read-only copy for scaling reads |
| **Endpoint** | DNS address to connect to database |
| **Parameter Group** | DB configuration settings |
| **Option Group** | Additional features (SSL, audit, etc.) |
| **Subnet Group** | Which subnets RDS can use |
| **Automated Backups** | Daily backup + transaction logs |
| **Snapshot** | Manual point-in-time backup |

---

## 💰 RDS Instance Classes

| Class | Use Case | Example |
|-------|----------|---------|
| **db.t3/t4g** | Dev/test, small apps | db.t3.micro (free tier!) |
| **db.m5/m6i** | General production | db.m5.large |
| **db.r5/r6g** | Memory-intensive | db.r5.large |
| **db.x2g** | SAP HANA, huge DBs | db.x2g.xlarge |

> Free tier: **db.t3.micro**, 20 GB storage, 20 GB backup

---

## 🛠️ Hands-On: Create RDS MySQL Instance

### Step 1: Create DB Subnet Group (required)
```
RDS → Subnet Groups → Create DB Subnet Group
  → Name: my-db-subnet-group
  → VPC: your-vpc
  → Add Subnets: Select private subnets from MULTIPLE AZs
    (ap-south-1a private subnet + ap-south-1b private subnet)
  → Create
```

### Step 2: Create Security Group for RDS
```
EC2 → Security Groups → Create
  → Name: rds-sg
  → VPC: your-vpc
  → Inbound Rules:
      Type: MySQL/Aurora (3306)
      Source: app-security-group (EC2's SG)
  → Create
```

### Step 3: Create RDS Instance
```
RDS → Databases → Create Database
  → Creation Method: Standard Create

Engine:
  → MySQL → Version: 8.0

Templates:
  → Free Tier (for learning)
  OR → Production (enables Multi-AZ, etc.)

Settings:
  → DB Instance Identifier: my-mysql-db
  → Master Username: admin
  → Master Password: MySecurePass123!

Instance Configuration:
  → DB Instance Class: db.t3.micro (free tier)

Storage:
  → Storage Type: gp2
  → Allocated Storage: 20 GiB
  → Storage Autoscaling: Enable (max 100 GiB)

Connectivity:
  → VPC: your-vpc
  → DB Subnet Group: my-db-subnet-group
  → Public Access: NO (private only!)
  → VPC Security Groups: rds-sg

Database Authentication:
  → Password authentication

Additional Configuration:
  → Initial Database Name: myappdb
  → Enable automated backups: Yes
  → Backup retention: 7 days

→ Create Database (takes 5-10 minutes)
```

### Step 4: Get RDS Endpoint
```
RDS → Databases → my-mysql-db
  → Connectivity & Security tab
  → Endpoint: my-mysql-db.abc123.ap-south-1.rds.amazonaws.com
  → Port: 3306
```

### Step 5: Connect from EC2
```bash
# Install MySQL client on EC2
sudo apt update
sudo apt install mysql-client -y

# Connect to RDS
mysql -h my-mysql-db.abc123.ap-south-1.rds.amazonaws.com \
      -P 3306 \
      -u admin \
      -p

# Enter password when prompted

# Inside MySQL:
SHOW DATABASES;
USE myappdb;
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
SELECT * FROM users;
```

---

## 🔄 Multi-AZ Deployment

Multi-AZ provides **high availability** with automatic failover:

```
Normal Operation:
  App → Primary DB (AZ-1) → Synchronous replication → Standby DB (AZ-2)
                                        (few ms lag)

Failover (AZ-1 fails):
  App → DNS endpoint (unchanged!) → AWS auto-promotes Standby
  Failover time: 1-2 minutes
  
Key Points:
  ✅ Same endpoint URL — no app changes needed
  ✅ Synchronous replication — zero data loss
  ✅ Automatic — no manual intervention
  ✅ Covers: AZ failure, DB crash, maintenance
  ❌ Standby is NOT readable (only for failover)
```

### Enable Multi-AZ
```
Create DB → Availability & Durability:
  → Multi-AZ DB instance: Create a standby instance

OR modify existing:
RDS → Database → Modify
  → Multi-AZ Deployment: Yes
  → Apply: Immediately or During next maintenance window
```

---

## 📖 Read Replicas

Read Replicas allow **scaling read traffic** across multiple copies:

```
Write Traffic:  App → Primary DB (only primary accepts writes)
Read Traffic:   App → Read Replica 1 (reports)
                App → Read Replica 2 (analytics)
                App → Read Replica 3 (different region)

Replication: Asynchronous (slight lag, usually <1 second)

Max Replicas: 15 (Aurora), 5 (MySQL/PostgreSQL/MariaDB)
```

### Create Read Replica
```
RDS → Select Primary DB → Actions → Create Read Replica
  → DB Instance Identifier: my-mysql-db-replica-1
  → Destination Region: ap-south-1 (same) or us-east-1 (cross-region!)
  → DB Instance Class: db.t3.medium
  → Multi-AZ: Optional
  → Create Read Replica

Connection:
  Replica endpoint: my-mysql-db-replica-1.xxx.ap-south-1.rds.amazonaws.com
  Port: 3306 (read-only connection)
```

---

## 🔁 Multi-AZ vs Read Replica

| Feature | Multi-AZ | Read Replica |
|---------|----------|-------------|
| Purpose | High Availability | Read Scaling |
| Replication | Synchronous | Asynchronous |
| Readable | ❌ No | ✅ Yes |
| Failover | Automatic | Manual (promote) |
| Region | Same region only | Cross-region possible |
| Use case | Production HA | Reporting, analytics |

---

## 💾 Backups

### Automated Backups
```
RDS automatically takes:
  - Daily full backup during backup window
  - Transaction logs every 5 minutes
  → Enables Point-in-Time Recovery (PITR) to any second!

Retention: 1-35 days (0 = disabled)
Storage: Free up to DB size

Backup Window:
  RDS → Database → Modify → Backup window
  → Set to low-traffic period (e.g., 02:00-03:00 UTC)
```

### Manual Snapshots
```
RDS → Database → Actions → Take Snapshot
  → Name: pre-upgrade-snapshot-2024-01
  → Take Snapshot

Snapshots:
  - Retained until manually deleted
  - Can copy to another region
  - Can share with other AWS accounts
  - Can restore to new DB instance
```

### Restore from Snapshot
```
RDS → Snapshots → Select → Restore Snapshot
  → New DB Identifier: my-db-restored
  → Instance Class, VPC, Security Group
  → Restore

OR Point-in-Time Restore:
RDS → Database → Actions → Restore to Point in Time
  → Choose: Custom date/time (any second within retention period!)
  → Restore DB Instance Identifier: my-db-pitr
```

---

## 🔐 RDS Security

### Encryption at Rest
```
Enable during creation:
  → Enable Encryption: Yes
  → KMS Key: aws/rds (default) or custom CMK

Note: Cannot enable encryption on existing unencrypted DB
Workaround: Snapshot → Copy with encryption → Restore
```

### Encryption in Transit (SSL/TLS)
```bash
# Connect with SSL
mysql -h mydb.xxx.rds.amazonaws.com \
      -u admin -p \
      --ssl-ca=rds-ca-2019-root.pem \
      --ssl-mode=REQUIRED

# Download RDS CA cert:
# https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

### IAM Authentication
```bash
# Generate auth token instead of password
TOKEN=$(aws rds generate-db-auth-token \
  --hostname mydb.xxx.rds.amazonaws.com \
  --port 3306 \
  --username iam-user \
  --region ap-south-1)

mysql -h mydb.xxx.rds.amazonaws.com -u iam-user \
      --password=$TOKEN --ssl-mode=REQUIRED
```

---

## ⚡ Aurora vs Standard RDS

| Feature | Aurora | MySQL/PostgreSQL RDS |
|---------|--------|---------------------|
| Performance | 5x MySQL, 3x Postgres | Standard |
| Storage | Auto-grows to 128 TB | Up to 64 TB |
| Replicas | Up to 15 | Up to 5 |
| Failover | ~30 seconds | 1-2 minutes |
| Cost | ~20% more | Standard |
| Serverless | ✅ Aurora Serverless v2 | ❌ |
| Global DB | ✅ Yes | ❌ No |

### Aurora Serverless v2
```
Auto-scales database capacity from 0.5 to 128 ACUs
  → Pay per ACU-second (0 cost when paused)
  → Perfect for dev/test or variable workloads

RDS → Create DB → Aurora → Serverless v2
  → Min ACUs: 0.5
  → Max ACUs: 16
```

---

## 🌍 RDS Proxy

Manages database connection pooling:

```
Without RDS Proxy:
  1000 Lambda functions → 1000 DB connections → DB overwhelmed!

With RDS Proxy:
  1000 Lambda functions → RDS Proxy (pools to 10-50 connections) → DB

RDS → Proxies → Create Proxy
  → Name: my-db-proxy
  → Engine: MySQL
  → DB Instance: my-mysql-db
  → Secrets: Store DB credentials in Secrets Manager
  → VPC: your-vpc
  → Create Proxy

Update app to use Proxy endpoint instead of DB endpoint
```

---

## 📊 RDS Monitoring

### CloudWatch Metrics
```
Key Metrics:
  CPUUtilization        → % CPU (alert if > 80%)
  DatabaseConnections   → Active connections
  FreeableMemory        → Available RAM
  FreeStorageSpace      → Disk space remaining
  ReadLatency           → Read response time
  WriteLatency          → Write response time
  ReplicaLag            → Read replica delay (seconds)
```

### Enhanced Monitoring
```
RDS → Modify → Monitoring → Enhanced Monitoring
  → Granularity: 1 second (vs 60s for basic)
  → Sends to CloudWatch Logs
  → Shows OS-level metrics: swap, processes, etc.
```

---

## 🔄 Parameter Groups

Customize database configuration:

```
RDS → Parameter Groups → Create Parameter Group
  → Family: mysql8.0
  → Name: my-mysql-params
  → Create

Edit parameters:
  → max_connections: 200
  → slow_query_log: 1
  → long_query_time: 2 (log queries > 2 seconds)
  → innodb_buffer_pool_size: {DBInstanceClassMemory*3/4}

Apply to DB:
  RDS → Database → Modify → Parameter Group → my-mysql-params
```

---

## ✅ Advantages

- **Fully managed** — AWS handles patching, backups, HA
- **Multi-AZ** — automatic failover
- **Automated backups** — PITR to any second
- **Read replicas** — horizontal read scaling
- **Multiple engines** — MySQL, Postgres, Oracle, SQL Server
- **Encryption** — at rest and in transit
- **Monitoring** — CloudWatch, Enhanced Monitoring, Performance Insights

## ❌ Disadvantages

- **No OS access** — cannot SSH into RDS
- **No custom plugins** — limited by managed service
- **Cost** — more expensive than self-managed DB on EC2
- **Size limits** — 64 TB max (use Aurora for more)
- **Single region** — Multi-AZ is same region (use Aurora Global for cross-region)

---

## 🏆 Best Practices

- Always put RDS in **private subnets** (never public!)
- Use **Multi-AZ** for all production databases
- **Encrypt at rest** — enable during creation
- Use **SSL/TLS** for all connections
- Set up **CloudWatch alarms** on CPU, connections, storage
- Use **Secrets Manager** for database credentials (not env vars)
- Enable **Performance Insights** for query analysis
- Configure **backup window** during off-peak hours
- Use **RDS Proxy** for Lambda + RDS (connection pooling)
- **Right-size** instances — start small, scale up

---

## 📚 Related Services

- **Aurora** — AWS-native cloud database (enhanced RDS)
- **RDS Proxy** — Connection pooling
- **Secrets Manager** — Manage DB credentials
- **Parameter Store** — Store config (less secure than Secrets Manager)
- **Performance Insights** — SQL-level performance analysis
- **DMS** — Database Migration Service (migrate to RDS)
