# 🪣 S3 - Simple Storage Service

## 📌 What is S3?

Amazon S3 (Simple Storage Service) is an **object storage service** that offers industry-leading scalability, data availability, security, and performance. You can store and retrieve any amount of data at any time.

> Think of S3 as **Google Drive/Dropbox for developers** — but infinitely scalable and deeply integrated with AWS.

---

## 🏗️ S3 Architecture

```
S3
└── Bucket (unique globally, region-specific)
    ├── Objects (Files)
    │   ├── Key (file name/path)
    │   ├── Value (file content)
    │   ├── Metadata (tags, content-type)
    │   └── Version ID (if versioning enabled)
    ├── Folders (prefix-based, not real folders)
    └── Policies (Bucket Policy, ACL)
```

---

## 📋 Key Concepts

| Term | Description |
|------|-------------|
| **Bucket** | Container for objects (like a folder) — globally unique name |
| **Object** | File stored in S3 (max 5TB per object) |
| **Key** | Unique identifier/path of an object in a bucket |
| **Region** | Bucket created in specific AWS region |
| **Versioning** | Keep multiple versions of same object |
| **ACL** | Access Control List — object-level permissions |
| **Bucket Policy** | JSON-based policy for bucket-level access |
| **Presigned URL** | Temporary URL to access private objects |

---

## 🗃️ S3 Storage Classes

| Class | Use Case | Retrieval | Cost |
|-------|----------|-----------|------|
| **S3 Standard** | Frequently accessed data | Immediate | High |
| **S3 Standard-IA** | Infrequently accessed | Immediate | Medium |
| **S3 One Zone-IA** | Non-critical, single AZ | Immediate | Low |
| **S3 Glacier Instant** | Archives, ms retrieval | Milliseconds | Lower |
| **S3 Glacier Flexible** | Archives, mins-hours | Minutes-Hours | Very Low |
| **S3 Glacier Deep Archive** | Long-term archives | 12-48 hours | Cheapest |
| **S3 Intelligent-Tiering** | Unknown access pattern | Immediate | Auto-tiered |

---

## 🌐 S3 Static Website Hosting

```
S3 Bucket
  └── Enable Static Website Hosting
      ├── Index Document: index.html
      └── Error Document: error.html

URL: http://bucket-name.s3-website.ap-south-1.amazonaws.com
```

---

## 🛠️ Hands-On: Create S3 Bucket & Host Static Website

### Step 1: Create Bucket
```
S3 → Create Bucket
  → Bucket Name: my-website-2024 (globally unique)
  → Region: ap-south-1
  → Create Bucket
```

### Step 2: Upload Files
```
Select Bucket → Objects → Upload
  → Add Files (index.html, style.css, images/)
  → Upload
```

### Step 3: Enable Public Access
```
Permissions Tab:
  → Block Public Access → Edit → UNCHECK all → Save
  → Object Ownership → ACLs Enabled → Save
  → Access Control List (ACL):
      → Everyone (public access): List ✓, Read ✓ → Save
```

### Step 4: Enable Static Hosting
```
Properties Tab:
  → Static Website Hosting → Enable
  → Index Document: index.html
  → Save Changes
```

### Step 5: Make Objects Public
```
Objects → Select All → Actions → Make Public Using ACL → Make Public
```

### Step 6: Access Website
```
Properties → Static Website Hosting → Copy URL → Paste in browser
```

---

## 🔄 S3 Replication

S3 Replication copies objects across buckets automatically.

### Types:
| Type | Description |
|------|-------------|
| **CRR** (Cross-Region Replication) | Replicate to different region |
| **SRR** (Same-Region Replication) | Replicate within same region |

### Hands-On: Setup Replication
```
Step 1: Create Source Bucket (versioning: ON)
Step 2: Create Destination Bucket (versioning: ON)
Step 3: Source Bucket → Management → Replication Rules → Create
  → Source: Apply to all objects
  → Destination: Choose destination bucket
  → IAM Role: Create new role
  → Save → Submit

Step 4: S3 Batch Operations (for existing objects)
  → Create Job → Source → Next → Copy Destination
  → IAM Role: Create custom role with proper policy
  → Run When Ready → Submit
```

---

## 🔐 S3 Security

### Bucket Policy (JSON Example)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### Access Control Methods
```
1. IAM Policies       → User/Role level access
2. Bucket Policies    → Bucket level access (JSON)
3. ACLs               → Object level (legacy)
4. Presigned URLs     → Temporary time-limited access
5. VPC Endpoints      → Private access within VPC
```

---

## 🔒 S3 Encryption

| Type | Description |
|------|-------------|
| **SSE-S3** | Server-side encryption with S3 keys |
| **SSE-KMS** | Server-side with AWS Key Management Service |
| **SSE-C** | Server-side with customer-provided keys |
| **Client-Side** | Encrypt before uploading |

---

## 📐 S3 Lifecycle Policies

Automatically transition objects between storage classes:

```
Standard → Standard-IA (after 30 days)
         → Glacier (after 90 days)
         → Delete (after 365 days)
```

---

## ✅ Advantages

- **Unlimited storage** — No capacity limit
- **99.999999999% (11 9s)** durability
- **Highly available** — 99.99% availability
- **Cost-effective** — Multiple storage tiers
- **Versioning** — Recover deleted/overwritten files
- **Event Notifications** — Trigger Lambda on upload
- **Global** — Works across regions

---

## ❌ Disadvantages

- Not suitable for database workloads
- High latency compared to EBS
- Eventually consistent (for some operations)
- Not a file system (no hierarchy, no locking)

---

## 🔄 S3 vs EBS vs EFS

| Feature | S3 | EBS | EFS |
|---------|----|-----|-----|
| Type | Object Store | Block Store | File System |
| Access | HTTP/API | EC2 only (same AZ) | Multiple EC2s |
| Max Size | Unlimited | 64 TB | Unlimited |
| Use Case | Files, backups | OS disk | Shared files |
| Cost | Lowest | Medium | Highest |

---

## 🏆 Best Practices

- Enable **versioning** on important buckets
- Use **lifecycle policies** to reduce costs
- Enable **MFA Delete** for critical data
- Use **S3 Transfer Acceleration** for global uploads
- Use **presigned URLs** for temporary access
- Enable **access logging** for audit trails
- Use **S3 Intelligent-Tiering** for unknown patterns
- Never make buckets public unless needed for static hosting

---

## 📚 Related Services

- **CloudFront** — CDN in front of S3 for global delivery
- **Lambda** — Trigger functions on S3 events
- **Athena** — Query S3 data with SQL
- **Glue** — ETL on S3 data
- **Replication** — Cross-region backup
