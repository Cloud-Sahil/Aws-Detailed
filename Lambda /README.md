# ⚡ AWS Lambda - Serverless Computing

## 📌 What is Lambda?

AWS Lambda is a **serverless compute service** that runs your code in response to events without provisioning or managing servers. You upload code, define a trigger, and AWS handles everything else — scaling, patching, availability.

> Think of Lambda as a **vending machine** — you don't own or manage the machine (server). You just put in code (money), press a trigger (button), and get a result. The machine handles the rest.

---

## 🧠 How Lambda Works (Internally)

```
Event Source (Trigger)
        ↓
   Lambda Service
        ↓
  [Cold Start: Download code → Init runtime → Start container]
        ↓ (only on first invoke or after idle)
  [Warm Start: Reuse existing container → Run handler]
        ↓
   Execute Function Code
        ↓
   Return Result / Write to destination
        ↓
   Container stays "warm" for ~15 minutes
```

---

## 🏗️ Lambda Architecture

```
                    ┌─────────────────────────────────────┐
                    │           AWS Lambda                │
                    │                                     │
Event Sources ───→  │  Trigger → Handler Function         │ ──→ Destinations
                    │           (your code)               │
                    │                                     │
                    │  Runtime: Python/Node/Java/Go/Ruby  │
                    │  Memory:  128 MB – 10,240 MB        │
                    │  Timeout: up to 15 minutes          │
                    └─────────────────────────────────────┘

Common Event Sources:          Common Destinations:
  ├── API Gateway               ├── S3
  ├── S3 (file upload)         ├── DynamoDB
  ├── DynamoDB Streams          ├── SNS / SQS
  ├── SQS                       ├── Another Lambda
  ├── CloudWatch Events         └── EventBridge
  ├── SNS
  ├── ALB
  └── Cognito
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **Handler** | Entry point function (e.g., `lambda_handler(event, context)`) |
| **Event** | JSON input data passed to function |
| **Context** | Runtime information (request ID, timeout, memory) |
| **Runtime** | Language environment (Python 3.12, Node.js 20.x, etc.) |
| **Deployment Package** | Your code + dependencies (ZIP or container image) |
| **Layer** | Shared libraries/dependencies across multiple functions |
| **Concurrency** | Number of simultaneous executions |
| **Cold Start** | Latency when Lambda initializes a new container |
| **Warm Start** | Reusing existing container (fast) |
| **Execution Role** | IAM Role granting Lambda permissions to AWS services |

---

## ⏱️ Lambda Limits

| Limit | Value |
|-------|-------|
| Max execution timeout | **15 minutes** |
| Memory | 128 MB – 10,240 MB (10 GB) |
| Deployment package size | 50 MB (ZIP direct), 250 MB (unzipped), 10 GB (container) |
| Ephemeral storage (/tmp) | 512 MB – 10,240 MB |
| Max concurrent executions | 1,000 per region (default, can increase) |
| Environment variables | 4 KB total |
| Payload (sync) | 6 MB request, 6 MB response |
| Payload (async) | 256 KB |

---

## 💻 Lambda Runtimes

| Runtime | Language | Version |
|---------|----------|---------|
| Python | Python | 3.8, 3.9, 3.10, 3.11, 3.12 |
| Node.js | JavaScript | 18.x, 20.x |
| Java | Java | 8, 11, 17, 21 |
| Go | Golang | provided.al2 |
| Ruby | Ruby | 3.2 |
| .NET | C# | 6, 8 |
| Custom | Any language | provided.al2023 |

---

## 💰 Lambda Pricing

| Metric | Price |
|--------|-------|
| Requests | $0.20 per 1 million requests |
| Duration | $0.0000166667 per GB-second |
| **Free Tier** | **1 million requests + 400,000 GB-seconds/month FOREVER** |

> Lambda free tier **never expires** (unlike EC2 12-month free tier)

### Cost Example
```
Function: 1M invocations/month, 512MB memory, 1 second each

Duration cost = 1,000,000 × 0.5 GB × 1 sec = 500,000 GB-seconds
Free tier:      400,000 GB-seconds
Billable:       100,000 GB-seconds × $0.0000166667 = $1.67

Request cost = 1,000,000 − 1,000,000 (free) = $0.00

Total: ~$1.67/month for 1 million executions!
```

---

## 🛠️ Hands-On: Create Your First Lambda Function

### Step 1: Create Lambda Function
```
AWS Console → Lambda → Create Function
  → Author from scratch
  → Function name: my-first-lambda
  → Runtime: Python 3.12
  → Architecture: x86_64
  → Execution Role: Create a new role with basic Lambda permissions
  → Create Function
```

### Step 2: Write Handler Code
```python
# Default handler in console editor
import json

def lambda_handler(event, context):
    print("Event received:", json.dumps(event))
    
    name = event.get('name', 'World')
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello, {name}!',
            'input': event
        })
    }
```

### Step 3: Deploy & Test
```
Code Tab → Deploy (saves code)

Test Tab → Create new test event:
  → Event name: test-hello
  → Event JSON:
    {
      "name": "AWS Learner"
    }
  → Save → Test

Output:
  Status: Succeeded
  Response:
    {
      "statusCode": 200,
      "body": "{\"message\": \"Hello, AWS Learner!\", ...}"
    }
```

### Step 4: View Logs
```
Monitor Tab → View CloudWatch Logs
  → Log Group: /aws/lambda/my-first-lambda
  → See: START, END, REPORT (duration, memory used)
```

---

## 🌐 Lambda + API Gateway (REST API)

Create a serverless API endpoint:

### Step 1: Create Lambda Function
```python
import json

def lambda_handler(event, context):
    http_method = event['httpMethod']
    path = event['path']
    body = json.loads(event.get('body') or '{}')
    
    if http_method == 'GET' and path == '/users':
        return {
            'statusCode': 200,
            'headers': {'Content-Type': 'application/json'},
            'body': json.dumps([
                {'id': 1, 'name': 'Alice'},
                {'id': 2, 'name': 'Bob'}
            ])
        }
    
    elif http_method == 'POST' and path == '/users':
        return {
            'statusCode': 201,
            'body': json.dumps({'message': 'User created', 'data': body})
        }
    
    return {'statusCode': 404, 'body': 'Not Found'}
```

### Step 2: Create API Gateway
```
API Gateway → Create API → REST API → Build
  → API Name: my-api
  → Create API

Create Resource:
  → /users

Create Methods:
  → GET → Lambda Function → my-lambda
  → POST → Lambda Function → my-lambda

Deploy API:
  → Actions → Deploy API
  → Stage: prod
  → Deploy

URL: https://abc123.execute-api.ap-south-1.amazonaws.com/prod/users
```

### Step 3: Test API
```bash
# GET request
curl https://abc123.execute-api.ap-south-1.amazonaws.com/prod/users

# POST request
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "Charlie"}' \
  https://abc123.execute-api.ap-south-1.amazonaws.com/prod/users
```

---

## 📁 Lambda + S3 Trigger (Auto-process uploads)

```python
import json
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    # Get bucket and file info from S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    
    print(f"New file uploaded: s3://{bucket}/{key}")
    
    # Process the file (example: read it)
    response = s3.get_object(Bucket=bucket, Key=key)
    content = response['Body'].read().decode('utf-8')
    
    print(f"File content: {content[:100]}")
    
    return {'statusCode': 200, 'body': 'Processed!'}
```

### Setup S3 Trigger
```
Lambda → Function → Configuration → Triggers → Add Trigger
  → S3
  → Bucket: my-bucket
  → Event type: PUT (All object create events)
  → Add

Now: Upload any file to my-bucket → Lambda auto-runs!
```

---

## 📦 Lambda Layers

Layers allow sharing code/dependencies across multiple functions:

```
Without Layers:                   With Layers:
Function A (50MB) ─ includes     Function A (1MB) ─┐
Function B (50MB) ─ same         Function B (1MB) ─┤→ Layer: numpy, pandas (50MB)
Function C (50MB) ─ libraries    Function C (1MB) ─┘

Lambda → Layers → Create Layer
  → Name: data-science-libs
  → Upload ZIP containing: numpy, pandas, scipy
  → Compatible runtimes: Python 3.12
  → Create

Attach to Function:
  Function → Code → Layers → Add Layer → Custom → data-science-libs
```

---

## 🔄 Lambda Concurrency

```
Default: 1,000 concurrent executions per region

Types:
  Reserved Concurrency: Max for this function (protects other functions)
  Provisioned Concurrency: Pre-warmed instances (eliminates cold start)

Example:
  Total account limit: 1,000
  Function A reserved: 500 (always available)
  Function B reserved: 300
  Remaining shared: 200
```

### Provisioned Concurrency (Eliminate Cold Starts)
```
Lambda → Function → Configuration → Concurrency
  → Provisioned concurrency: 10
  → AWS keeps 10 warm instances ready
  → ~$0.015/hour per provisioned instance
```

---

## ❄️ Cold Start vs Warm Start

```
Cold Start (slow, ~100ms - 5000ms):
  New request → Pull container image → Start runtime → Init code → Run handler

Warm Start (fast, ~1ms - 50ms):
  New request → Reuse running container → Run handler

Cold Start Reduction Tips:
  ✅ Use Provisioned Concurrency
  ✅ Use smaller deployment packages
  ✅ Choose faster runtimes (Node.js, Python > Java)
  ✅ Keep Lambda in /tmp for reuse
  ✅ Avoid large dependencies
  ✅ Use Lambda SnapStart (Java)
```

---

## 🔐 Lambda Security

### Execution Role (IAM)
```
Lambda needs IAM Role to access other AWS services:

Example Role Policies:
  AWSLambdaBasicExecutionRole  → CloudWatch Logs (minimum)
  AmazonS3ReadOnlyAccess       → Read S3
  AmazonDynamoDBFullAccess     → Read/write DynamoDB

Best Practice: Create custom minimal policy
{
  "Effect": "Allow",
  "Action": ["s3:GetObject"],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

### Lambda in VPC
```
Lambda → Configuration → VPC → Edit
  → VPC: my-vpc
  → Subnets: private subnets (multiple AZs!)
  → Security Group: lambda-sg

Use case: Lambda needs to access RDS (in private subnet)

Note: Lambda in VPC has NO internet by default
      Add NAT Gateway for internet access from VPC Lambda
```

---

## 📊 Lambda Destinations

Configure where to send results of async invocations:

```
Lambda → Configuration → Destinations → Add
  → On Success: SQS queue, SNS topic, EventBridge, another Lambda
  → On Failure: SQS queue (Dead Letter Queue), SNS
```

### Dead Letter Queue (DLQ)
```
Failed async invocations → SQS DLQ for retry/debugging

Lambda → Configuration → Asynchronous Invocation
  → Dead letter queue service: SQS
  → Queue: my-dlq
  → Save
```

---

## 🔄 Lambda vs EC2 vs ECS

| Feature | Lambda | EC2 | ECS/Fargate |
|---------|--------|-----|-------------|
| Server management | None | Full | Partial |
| Scaling | Automatic | Manual/Auto | Automatic |
| Max runtime | 15 min | Unlimited | Unlimited |
| Cold start | Yes | No | Yes |
| Cost model | Per invocation | Per hour | Per task |
| Use case | Short tasks, events | Long-running apps | Containers |
| Memory max | 10 GB | TB+ | 120 GB |

---

## 🛠️ Lambda Environment Variables

```
Lambda → Configuration → Environment Variables → Edit
  → Key: DB_HOST    Value: mydb.cluster.ap-south-1.rds.amazonaws.com
  → Key: DB_PORT    Value: 5432
  → Key: ENV        Value: production
  → Save

Access in code (Python):
import os
db_host = os.environ['DB_HOST']
```

---

## 📈 Lambda Monitoring Metrics

| Metric | Description |
|--------|-------------|
| Invocations | Total function calls |
| Duration | Execution time (ms) |
| Errors | Failed executions |
| Throttles | Concurrency limit exceeded |
| ConcurrentExecutions | Simultaneous runs |
| IteratorAge | SQS/Kinesis lag |

---

## ✅ Advantages

- **Zero server management** — AWS handles everything
- **Auto-scaling** — from 0 to thousands instantly
- **Pay per use** — only charged for execution time
- **Event-driven** — reacts to any AWS event
- **Multiple runtimes** — Python, Node.js, Java, Go, etc.
- **Integrated** with all AWS services
- **Free tier never expires** — 1M requests/month free

## ❌ Disadvantages

- **15-minute max timeout** — not for long-running tasks
- **Cold starts** — latency on first invocation
- **Stateless** — no persistent connections between invocations
- **Limited resources** — 10GB RAM, 10GB /tmp max
- **Debugging complexity** — harder than traditional servers
- **Vendor lock-in** — tied to AWS event model

---

## 🏆 Best Practices

- **Single responsibility** — one function, one job
- **Keep packages small** — faster cold starts
- **Use Layers** for shared dependencies
- **Set appropriate timeout** — don't use 15 min as default
- **Use environment variables** for configuration (not hardcoded)
- **Always use IAM roles** — never access keys
- **Enable X-Ray tracing** for debugging
- **Use Lambda Power Tuning** to find optimal memory/cost
- **Use Provisioned Concurrency** for latency-sensitive functions
- **Set up DLQ** for async invocations

---

## 📚 Related Services

- **API Gateway** — HTTP endpoint for Lambda
- **EventBridge** — Event routing to Lambda
- **SQS** — Queue-based trigger
- **S3** — File upload trigger
- **DynamoDB Streams** — DB change trigger
- **Step Functions** — Orchestrate multiple Lambdas
- **SAM** — Serverless Application Model (IaC for Lambda)
