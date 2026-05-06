# 🔐 IAM - Identity & Access Management

## 📌 What is IAM?

AWS IAM (Identity & Access Management) is a service that helps you **securely control access** to AWS resources. It lets you manage who (authentication) can do what (authorization) in your AWS account.

> IAM is the **security gatekeeper** of AWS — zero-cost, global service that controls all access.

---

## 🏗️ IAM Architecture

```
AWS Account (Root)
└── IAM
    ├── Users          ← Individual people
    │   └── Access Keys / Console Password
    ├── Groups         ← Collection of users
    │   └── Policies attached to group
    ├── Roles          ← Temporary permissions for services/users
    │   └── Trust Policy + Permission Policy
    └── Policies       ← JSON documents defining permissions
        ├── AWS Managed Policies
        ├── Customer Managed Policies
        └── Inline Policies
```

---

## 📋 IAM Core Components

| Component | Description | Example |
|-----------|-------------|---------|
| **User** | Individual identity (human or service) | john@company.com |
| **Group** | Collection of users sharing permissions | Developers, Admins |
| **Role** | Temporary credential for services | EC2 access to S3 |
| **Policy** | JSON document defining permissions | Allow S3:GetObject |
| **Access Key** | Programmatic access (CLI/SDK) | AKIAIOSFODNN7EXAMPLE |
| **MFA** | Multi-Factor Authentication | Google Authenticator |

---

## 📄 IAM Policy Structure (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-south-1"
        }
      }
    }
  ]
}
```

### Policy JSON Fields Explained
| Field | Description |
|-------|-------------|
| `Version` | Policy language version (always use 2012-10-17) |
| `Statement` | Array of permission rules |
| `Sid` | Statement ID (optional label) |
| `Effect` | Allow or Deny |
| `Action` | AWS API calls (e.g., s3:GetObject) |
| `Resource` | ARN of what the action applies to |
| `Condition` | Extra conditions (IP, time, MFA, etc.) |

---

## 🛠️ Hands-On: Create IAM User

### Step 1: Create User
```
IAM → Users → Create User
  → Username: dev-user-01
  → Check: Provide user access to AWS Management Console
  → Console Password: Custom → Set password
  → Uncheck: User must create new password at next sign-in
  → Next → Next → Create User
```

### Step 2: Create Group & Assign Policy
```
IAM → User Groups → Create Group
  → Group Name: Developers
  → Attach Permissions Policies:
      → AmazonS3FullAccess
      → AmazonEC2ReadOnlyAccess
  → Create Group

IAM → Users → dev-user-01 → Groups → Add to Group → Developers
```

### Step 3: Get Console Sign-in Link
```
IAM → Dashboard → Account ID / Sign-in URL
  → Copy: https://123456789.signin.aws.amazon.com/console
  → Share with user + username + password
```

---

## 🎭 IAM Roles

Roles provide **temporary credentials** to AWS services, users, or external identities.

### Common Use Cases
| Role Use Case | Example |
|---------------|---------|
| EC2 → S3 | EC2 instance reads from S3 without access keys |
| Lambda → DynamoDB | Lambda function writes to DB |
| Cross-account | User in Account A accesses Account B |
| Federated | Company SSO login to AWS |

### Hands-On: Create Role for EC2 → CloudWatch
```
IAM → Roles → Create Role
  → Trusted Entity: AWS Service
  → Use Case: EC2
  → Next

  → Add Permissions: CloudWatchAgentServerPolicy
  → Next

  → Role Name: ec2-cloudwatch-role
  → Create Role

Attach to EC2:
  EC2 → Instance → Actions → Security → Modify IAM Role
  → Select: ec2-cloudwatch-role → Update
```

---

## 🔑 Access Keys (CLI/Programmatic Access)

```
IAM → Users → Select User → Security Credentials
  → Create Access Key
  → Use Case: CLI
  → Download .csv (save it — shown only once!)

Configure AWS CLI:
$ aws configure
  AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
  AWS Secret Access Key: your-secret-key
  Default region: ap-south-1
  Default output format: json
```

---

## 🔒 Password Policy

```
IAM → Account Settings → Password Policy → Edit
  → Minimum length: 12
  → Require uppercase, lowercase, numbers, symbols
  → Allow users to change their password
  → Password expiration: 90 days
  → Prevent password reuse: 5 passwords
```

---

## 🔢 IAM Policy Types Comparison

| Type | Managed By | Reusable | Use Case |
|------|-----------|----------|----------|
| **AWS Managed** | AWS | Yes | Common permissions (S3, EC2, etc.) |
| **Customer Managed** | You | Yes | Custom permissions for your org |
| **Inline** | Attached to entity | No | One-off, specific permissions |

---

## 🔐 IAM Best Practices

### ✅ DO:
- **Lock root account** — Enable MFA, don't use daily
- **Use groups** for permissions, not individual users
- **Use roles** for applications/services (not access keys on EC2)
- **Apply least privilege** — Give minimum required permissions
- **Rotate access keys** regularly (or use roles instead)
- **Enable MFA** for all console users
- **Use IAM Access Analyzer** to review permissions

### ❌ DON'T:
- Don't share root account credentials
- Don't hardcode access keys in code
- Don't give AdministratorAccess to everyone
- Don't create access keys unless absolutely necessary

---

## 🌍 IAM is Global

IAM is a **global service** — users, groups, and roles are available across all AWS regions. No region selection needed.

---

## 🔄 IAM vs Resource-Based Policies

| Feature | IAM Policy | Resource Policy |
|---------|-----------|----------------|
| Attached to | User/Group/Role | Resource (S3, SQS) |
| Cross-account | With role | Directly possible |
| Example | Allow EC2 access | S3 bucket policy |

---

## 📊 ARN Format

```
arn:partition:service:region:account-id:resource-type/resource-id

Examples:
arn:aws:iam::123456789012:user/john
arn:aws:s3:::my-bucket
arn:aws:ec2:ap-south-1:123456789012:instance/i-1234567890abcdef0
arn:aws:iam::123456789012:role/my-role
```

---

## ✅ Advantages

- Centralized access control
- Fine-grained permissions
- No additional cost
- Integrates with all AWS services
- Audit via CloudTrail

---

## ❌ Disadvantages

- Complex policy management at scale
- IAM policy limits (10 managed policies per user/group/role)
- Policy evaluation can be confusing (explicit Deny > Allow)

---

## 📚 Related Services

- **CloudTrail** — Audit who did what in IAM
- **AWS SSO / IAM Identity Center** — Federated login
- **AWS Organizations** — Centrally manage multiple accounts
- **AWS STS** — Security Token Service (temporary credentials)
