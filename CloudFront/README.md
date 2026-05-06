# 🌍 CloudFront - Content Delivery Network (CDN)

## 📌 What is CloudFront?

Amazon CloudFront is a **Content Delivery Network (CDN)** that delivers content (web pages, images, videos, APIs) to users globally with low latency by caching at **edge locations** worldwide.

> Think of CloudFront as having **copies of your website in 400+ locations worldwide** — users get content from the nearest location, not from a single server.

---

## 🏗️ CloudFront Architecture

```
User in Mumbai
      ↓
Edge Location (Mumbai - nearest)
      ├── Cache HIT → Return cached content (fast! ~ms)
      └── Cache MISS → Fetch from Origin → Cache → Return

Origins:
  ├── S3 Bucket
  ├── EC2 Instance
  ├── Application Load Balancer
  └── Custom HTTP Server (on-premise)

Edge Locations: 400+ worldwide
Regional Edge Caches: 13 (between edge & origin)
```

---

## 📋 Key Concepts

| Concept | Description |
|---------|-------------|
| **Distribution** | CloudFront configuration for your content |
| **Edge Location** | Server where content is cached (400+ globally) |
| **Origin** | Source of original content (S3, EC2, ALB) |
| **Behavior** | Rules for caching, protocols, headers |
| **TTL** | How long content is cached (Time To Live) |
| **Invalidation** | Force delete cached content |
| **OAC** | Origin Access Control — S3 private with CF |
| **WAF** | Web Application Firewall integration |

---

## 🌐 CloudFront + S3 Setup

### Step 1: Setup S3 Static Website
```
S3 → Create Bucket
  → Block Public Access: OFF
  → Object Ownership: ACLs Enabled
  → Upload: index.html, images/
  → ACL: Everyone → List, Read
  → Properties → Static Website Hosting → Enable
  → Index Document: index.html
```

### Step 2: Create CloudFront Distribution
```
CloudFront → Create Distribution
  → Origin Domain: Select your S3 bucket
  → Origin Access: Public (for public S3)
  OR → Origin Access Control (OAC) for private S3

Next:
  → WAF: Do not enable (for free tier testing)

Next:
  → Default Root Object: index.html
  → Create Distribution

Wait: ~5-10 minutes for deployment
```

### Step 3: Configure Behaviors
```
Distribution → Behaviors → Select → Edit
  → Viewer Protocol Policy: Redirect HTTP to HTTPS
  → Allowed HTTP Methods: GET, HEAD
  → Cache Policy: CachingOptimized
  → Save Changes
```

### Step 4: Configure Origin Settings
```
Distribution → Origins → Select → Edit
  → Protocol: Match Viewer (or HTTPS only)
  → Save Changes
```

### Step 5: Access via CloudFront URL
```
Distribution → Domain Name: d1234abcd.cloudfront.net
Open: https://d1234abcd.cloudfront.net
```

---

## 🔒 CloudFront + S3 Private (OAC)

Keep S3 private, only allow CloudFront to access:

```
S3:
  → Block ALL public access: ON
  → No public ACL

CloudFront:
  → Origin Access: Create Control Setting (OAC)
  → Select OAC → Create

S3 automatically gets bucket policy allowing only CloudFront:
{
  "Effect": "Allow",
  "Principal": {"Service": "cloudfront.amazonaws.com"},
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT:distribution/DIST_ID"
    }
  }
}
```

---

## 🔑 Cache Behaviors & TTL

```
Default TTL: 24 hours (86400 seconds)
Minimum TTL: 0 seconds
Maximum TTL: 365 days

Override with headers:
  Cache-Control: max-age=3600    (1 hour)
  Cache-Control: no-cache        (always revalidate)
  Expires: Thu, 01 Jan 2025...
```

### Invalidation (Clear Cache)
```
Distribution → Invalidations → Create Invalidation
  → Object paths: /*          (clear everything)
  → Object paths: /images/*   (clear images only)
  → Object paths: /index.html (clear specific file)
  → Create Invalidation

Cost: First 1000 invalidation paths/month free
```

---

## 🌐 Custom Domain with CloudFront

```
1. Request SSL certificate in ACM (us-east-1 ONLY for CF!)
   ACM → Request → Public → domain.com, *.domain.com

2. CloudFront Distribution → Edit
   → Alternate domain names: www.myapp.com
   → Custom SSL Certificate: Select from ACM
   → Save

3. Route 53 → A Record (ALIAS)
   → Name: www.myapp.com
   → Value: d1234abcd.cloudfront.net
```

---

## 🛡️ WAF Integration

```
CloudFront → WAF → AWS WAF → Enable
  → Create/Select Web ACL
  → Rules: SQL injection, XSS, Rate limiting
  → Associate with distribution
```

---

## 🔄 CloudFront Functions vs Lambda@Edge

| Feature | CF Functions | Lambda@Edge |
|---------|-------------|-------------|
| Location | Edge | Regional Edge Cache |
| Latency | Ultra-low | Low |
| Runtime | JS only | Node.js, Python |
| Triggers | Viewer request/response | All 4 events |
| Max duration | 1ms | 5s (viewer), 30s (origin) |
| Use case | URL rewrite, header mod | Auth, A/B testing |

---

## 📊 CloudFront Metrics (CloudWatch)

| Metric | Description |
|--------|-------------|
| Requests | Total requests |
| BytesDownloaded | Data transferred |
| CacheHitRate | % of requests from cache |
| 4xxErrorRate | Client errors |
| 5xxErrorRate | Origin errors |
| OriginLatency | Time to fetch from origin |

---

## ✅ Advantages

- **Global performance** — content from nearest edge
- **Reduces origin load** — cached content doesn't hit origin
- **DDoS protection** — AWS Shield Standard included
- **HTTPS** — Easy SSL with ACM (free)
- **Cost-effective** — Less origin requests = lower cost
- **Programmable** — CF Functions, Lambda@Edge

## ❌ Disadvantages

- **Cache staleness** — Changes may not appear immediately
- **Invalidation cost** — Many invalidations can be expensive
- **Debugging** — Hard to trace issues across edge locations
- **Not for APIs** — Dynamic content doesn't benefit from caching

## 🏆 Best Practices

- Use **OAC** to keep S3 private
- Set appropriate **TTL values** per content type
- Use **Cache-Control headers** from origin
- Enable **CloudFront access logs** to S3
- Use **Geo-restriction** to block unwanted countries
- Enable **compression** (gzip, Brotli) automatically
- Use **Price Class** to limit to specific regions (cost savings)

---

## 📚 Related Services

- **S3** — Primary origin for static content
- **ACM** — Free SSL certificates (must be us-east-1)
- **WAF** — Web application firewall
- **Route 53** — DNS pointing to CloudFront
- **Shield** — DDoS protection (Standard free, Advanced paid)
