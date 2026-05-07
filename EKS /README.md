# 🐳 EKS - Elastic Kubernetes Service

## 📌 What is EKS?

Amazon EKS (Elastic Kubernetes Service) is a **fully managed Kubernetes service** that makes it easy to deploy, manage, and scale containerized applications using Kubernetes on AWS without needing to install or operate your own Kubernetes control plane.

> Think of EKS as **managed Kubernetes** — AWS runs the Kubernetes control plane (brain), you focus on running your containers (workloads).

---

## 🧠 Kubernetes Basics (Prerequisites)

Before EKS, understand these Kubernetes concepts:

```
Kubernetes Cluster
├── Control Plane (managed by AWS in EKS)
│   ├── API Server      ← Entry point for all K8s commands
│   ├── etcd            ← Database storing cluster state
│   ├── Scheduler       ← Assigns pods to nodes
│   └── Controller Mgr  ← Maintains desired state
│
└── Worker Nodes (you manage these)
    ├── Node 1 (EC2 instance)
    │   ├── Pod A  [Container 1, Container 2]
    │   ├── Pod B  [Container 3]
    │   └── kubelet (node agent)
    ├── Node 2 (EC2 instance)
    │   └── Pod C  [Container 4]
    └── Node 3 (EC2 instance)
        └── Pod D  [Container 5]
```

---

## 📋 Kubernetes Key Objects

| Object | Description | Example |
|--------|-------------|---------|
| **Pod** | Smallest unit, runs 1+ containers | nginx pod |
| **Deployment** | Manages pods, rolling updates, replicas | 3 replicas of nginx |
| **Service** | Exposes pods (ClusterIP, NodePort, LoadBalancer) | nginx-service:80 |
| **Namespace** | Virtual cluster within cluster | dev, staging, prod |
| **ConfigMap** | Non-sensitive configuration | DB_HOST=localhost |
| **Secret** | Sensitive configuration (base64 encoded) | DB_PASSWORD |
| **Ingress** | HTTP routing rules (like ALB rules) | /api → api-service |
| **PersistentVolume** | Storage resource | 10Gi EBS volume |
| **HPA** | Horizontal Pod Autoscaler | scale pods on CPU |
| **DaemonSet** | One pod per node | log collector |

---

## 🏗️ EKS Architecture

```
AWS Cloud
└── EKS Cluster: my-cluster
    │
    ├── Control Plane (AWS Managed — you don't see these)
    │   ├── API Server (endpoint: xxx.eks.amazonaws.com)
    │   ├── etcd
    │   ├── Scheduler
    │   └── Controller Manager
    │
    └── Data Plane (Worker Nodes — you manage)
        ├── Node Group 1: On-Demand (m5.large × 3)
        │   ├── Pod: frontend (nginx)
        │   ├── Pod: backend (node.js)
        │   └── System pods (CoreDNS, kube-proxy)
        │
        ├── Node Group 2: Spot (m5.large × 5) — cost savings
        │
        └── Fargate Profile (serverless — no EC2 to manage)
            └── Pod: batch-job (runs on demand)
```

---

## 🔄 EKS Worker Node Types

| Type | Description | Management | Cost |
|------|-------------|-----------|------|
| **Managed Node Groups** | AWS manages EC2 lifecycle | Low | Medium |
| **Self-Managed Nodes** | You manage EC2 fully | High | Low |
| **Fargate** | Serverless pods, no EC2 | None | Higher per pod |

---

## 🛠️ Hands-On: Create EKS Cluster

### Prerequisites: Install Tools
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip && sudo ./aws/install
aws --version

# Install kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
kubectl version --client

# Install eksctl (EKS CLI tool)
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Method 1: Create Cluster via eksctl (Recommended)
```bash
# Create cluster with managed node group
eksctl create cluster \
  --name my-eks-cluster \
  --region ap-south-1 \
  --nodegroup-name my-nodes \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 5 \
  --managed

# Takes ~15-20 minutes
# Automatically updates ~/.kube/config
```

### Method 2: Create Cluster via Console
```
EKS → Clusters → Create
  → Name: my-eks-cluster
  → K8s Version: 1.29
  → Cluster IAM Role: Create new (eksClusterRole)
  → VPC: default or custom
  → Subnets: Multiple AZs
  → Security Group: default
  → Next → Create

Add Node Group:
  → Cluster → Compute → Add Node Group
  → Name: my-nodes
  → Node IAM Role: Create new (eksNodeRole)
  → AMI: Amazon Linux 2
  → Instance type: t3.medium
  → Scaling: min=2, desired=3, max=5
  → Create
```

### Connect kubectl to Cluster
```bash
# Update kubeconfig
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name my-eks-cluster

# Verify connection
kubectl get nodes
# NAME                                      STATUS   ROLES    AGE
# ip-192-168-1-100.ap-south-1.compute.internal   Ready    <none>   5m
```

---

## 🚀 Deploy an Application on EKS

### Step 1: Create Deployment YAML
```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # 3 pods
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25        # Docker image
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Step 2: Create Service YAML (Expose App)
```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer    # Creates AWS ALB/ELB automatically!
```

### Step 3: Apply & Verify
```bash
# Deploy
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml

# Check pods
kubectl get pods
# NAME                                READY   STATUS    RESTARTS   AGE
# nginx-deployment-7d6b4c5f8-abc12   1/1     Running   0          1m
# nginx-deployment-7d6b4c5f8-def34   1/1     Running   0          1m
# nginx-deployment-7d6b4c5f8-ghi56   1/1     Running   0          1m

# Check service & get external IP
kubectl get services
# NAME            TYPE           CLUSTER-IP     EXTERNAL-IP              PORT(S)
# nginx-service   LoadBalancer   10.100.0.123   a1b2c3.elb.amazonaws.com 80:30001/TCP

# Access app
curl http://a1b2c3.elb.amazonaws.com
```

---

## 📈 Horizontal Pod Autoscaler (HPA)

Auto-scale pods based on CPU/memory:

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # Scale when CPU > 70%
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
```

---

## 🗂️ Namespaces

```bash
# Create namespaces for environments
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

# Deploy to specific namespace
kubectl apply -f nginx-deployment.yaml -n dev
kubectl get pods -n dev
kubectl get pods --all-namespaces
```

---

## 🔐 EKS IAM Integration (IRSA)

IAM Roles for Service Accounts — give pods AWS permissions:

```bash
# Enable OIDC provider for cluster
eksctl utils associate-iam-oidc-provider \
  --cluster my-eks-cluster \
  --approve

# Create IAM service account with S3 access
eksctl create iamserviceaccount \
  --cluster my-eks-cluster \
  --namespace default \
  --name s3-access-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

```yaml
# Use in pod:
spec:
  serviceAccountName: s3-access-sa
  containers:
  - name: my-app
    image: my-app:latest
    # Pod now has S3 read access via IAM Role!
```

---

## 💾 Persistent Storage in EKS (EBS CSI Driver)

```bash
# Install EBS CSI driver
eksctl create addon \
  --name aws-ebs-csi-driver \
  --cluster my-eks-cluster
```

```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp3

---
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 10Gi

---
# Use in pod:
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - name: app
    volumeMounts:
    - mountPath: /data
      name: data
```

---

## 🌐 EKS Ingress (AWS Load Balancer Controller)

```bash
# Install AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-eks-cluster
```

```yaml
# ingress.yaml — path-based routing
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

---

## 🔧 Essential kubectl Commands

```bash
# ──── Cluster Info ────
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide           # Show IPs, OS

# ──── Pods ────
kubectl get pods
kubectl get pods -n kube-system     # System pods
kubectl get pods -o wide            # Show which node
kubectl describe pod <pod-name>     # Full details
kubectl logs <pod-name>             # View logs
kubectl logs <pod-name> -f          # Stream logs
kubectl exec -it <pod-name> -- bash # Shell into pod
kubectl delete pod <pod-name>

# ──── Deployments ────
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl rollout status deployment/nginx-deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment   # Rollback!

# ──── Services ────
kubectl get services
kubectl get svc

# ──── Apply/Delete ────
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl apply -f .    # Apply all YAML in directory

# ──── Debugging ────
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods
kubectl top nodes
```

---

## 🔄 EKS vs ECS vs Lambda

| Feature | EKS | ECS | Lambda |
|---------|-----|-----|--------|
| Type | Managed K8s | Managed Containers | Serverless |
| Control | High | Medium | Low |
| Portability | High (K8s standard) | AWS-specific | AWS-specific |
| Complexity | High | Medium | Low |
| Cold start | No | No | Yes |
| Max runtime | Unlimited | Unlimited | 15 min |
| Cost | Control plane $0.10/hr | Free | Per invocation |
| Use case | Complex microservices | Simpler containers | Event-driven |

---

## ✅ Advantages

- **Fully managed** control plane (AWS patches, updates)
- **Standard Kubernetes** — portable, not vendor-locked
- **Auto-scaling** — nodes and pods
- **Multi-AZ** — high availability built-in
- **Integration** — ALB, IAM, EBS, EFS, CloudWatch
- **Fargate support** — serverless option
- **IRSA** — fine-grained IAM per pod

## ❌ Disadvantages

- **Complexity** — steep Kubernetes learning curve
- **Cost** — Control plane: $0.10/hour ($72/month minimum)
- **Overkill** — for simple apps, ECS or Lambda is easier
- **Networking complexity** — VPC CNI, security groups per pod

---

## 🏆 Best Practices

- Use **Managed Node Groups** — AWS handles node updates
- **Multi-AZ** node groups for HA
- Use **IRSA** (not instance profiles) for pod-level IAM
- Set **resource requests and limits** on all pods
- Use **namespaces** to separate environments
- Enable **cluster logging** to CloudWatch
- Use **HPA + Cluster Autoscaler** for dynamic scaling
- Use **Spot instances** in non-critical node groups (70-90% savings)
- **Tag** all AWS resources created by EKS

---

## 🧹 Cleanup (Avoid Costs!)

```bash
# Delete all resources before deleting cluster
kubectl delete all --all
kubectl delete pvc --all

# Delete cluster (deletes all node groups + resources)
eksctl delete cluster --name my-eks-cluster --region ap-south-1
```

---

## 📚 Related Services

- **ECR** — Elastic Container Registry (store Docker images)
- **ECS** — Simpler container service (EKS alternative)
- **Fargate** — Serverless compute for EKS/ECS
- **AWS Load Balancer Controller** — ALB for Kubernetes
- **CloudWatch Container Insights** — EKS monitoring
- **App Mesh** — Service mesh for EKS
