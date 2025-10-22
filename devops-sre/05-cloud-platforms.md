# 5. Cloud Platforms (AWS/Azure/GCP)

## 📚 Quick Summary

Cloud platforms are where everything runs - master AWS, Azure, and GCP essentials!

**What You'll Learn:**
- **AWS**: EC2, ECS, EKS, Lambda, S3, RDS, CloudWatch, IAM, VPC
- **Azure**: VMs, AKS, Functions, Blob Storage, Cognitive Services
- **GCP**: Compute Engine, GKE, Cloud Run, Cloud Storage, Vertex AI
- **Networking**: VPCs, Subnets, Security Groups, Load Balancers
- **IAM**: Identity & Access Management, Roles, Policies
- **Storage**: Object storage, Block storage, File storage
- **Managed Services**: Databases, Kubernetes, Serverless

**Why This Matters:**
- 95% of production workloads run on cloud
- AWS = 32% market share (#1)
- Multi-cloud strategy increasingly common
- Cloud certifications = $20k-30k salary boost
- Interview weight: 10-15% of questions

**Interview Reality:**
"How would you deploy this on AWS?" = Daily DevOps question!

---

## 📖 Simple Explanation

**What is Cloud?**
Rent computers, storage, and services instead of buying them.

**Before Cloud:**
```
Company wants web app:
1. Buy servers ($10k-$100k)
2. Set up data center
3. Hire ops team
4. Wait weeks/months
5. If wrong size? Wasted money

Problems: Expensive, slow, inflexible
```

**With Cloud:**
```
1. Click "Create server" (seconds)
2. Pay by the hour ($0.10/hour)
3. Scale up/down instantly
4. No upfront cost
5. Global in minutes

Benefits: Fast, cheap, flexible, pay-as-you-go
```

**Cloud Service Models:**
```
IaaS (Infrastructure):
You manage: OS, apps, data
Cloud manages: Servers, networking, storage
Example: EC2, Azure VMs

PaaS (Platform):
You manage: Apps, data
Cloud manages: OS, runtime, scaling
Example: App Engine, Azure App Service

SaaS (Software):
You manage: Data
Cloud manages: Everything
Example: Gmail, Salesforce

Containers/Kubernetes:
You manage: Containers, apps
Cloud manages: K8s control plane, nodes
Example: EKS, AKS, GKE
```

---

## Table of Contents
- [AWS (Amazon Web Services)](#aws-amazon-web-services)
- [Azure (Microsoft)](#azure-microsoft)
- [GCP (Google Cloud Platform)](#gcp-google-cloud-platform)
- [Cloud Networking](#cloud-networking)
- [IAM & Security](#iam--security)
- [Storage Options](#storage-options)
- [Managed Kubernetes](#managed-kubernetes)
- [Serverless](#serverless)
- [Cost Optimization](#cost-optimization)
- [Multi-Cloud Strategy](#multi-cloud-strategy)

---

## AWS (Amazon Web Services)

### 📖 Simple Explanation

**AWS = Biggest cloud provider**
- Started 2006 (first to market)
- 200+ services
- 32% market share
- Used by: Netflix, Airbnb, NASA

**Core Services You MUST Know:**

---

### EC2 (Elastic Compute Cloud)

**Virtual machines in the cloud**

**Instance Types:**
```
t3.micro    - 1 vCPU, 1GB RAM  ($0.01/hr) - Dev/test
t3.medium   - 2 vCPU, 4GB RAM  ($0.04/hr) - Small apps
c5.xlarge   - 4 vCPU, 8GB RAM  ($0.17/hr) - Compute-heavy
m5.2xlarge  - 8 vCPU, 32GB RAM ($0.38/hr) - Balanced
r5.4xlarge  - 16 vCPU, 128GB RAM ($1.01/hr) - Memory-heavy
p3.2xlarge  - 8 vCPU, 61GB RAM, 1 GPU ($3.06/hr) - ML/AI
```

**Launch EC2 Instance (AWS CLI):**
```bash
# Create key pair
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 400 my-key.pem

# Launch instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \  # Ubuntu 22.04
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server}]'

# List instances
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running"

# SSH into instance
ssh -i my-key.pem ubuntu@<public-ip>

# Stop instance
aws ec2 stop-instances --instance-ids i-0123456789abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

---

### EKS (Elastic Kubernetes Service)

**Managed Kubernetes**

```bash
# Install eksctl
curl --silent --location "https://github.com/weksctl-io/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create EKS cluster
eksctl create cluster \
  --name my-cluster \
  --region us-west-2 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 10 \
  --managed

# Takes ~15 minutes

# Configure kubectl
aws eks update-kubeconfig --region us-west-2 --name my-cluster

# Deploy application
kubectl create deployment nginx --image=nginx:latest --replicas=3
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Get LoadBalancer URL
kubectl get svc nginx

# Delete cluster
eksctl delete cluster --name my-cluster
```

---

### S3 (Simple Storage Service)

**Object storage (files, backups, static websites)**

```bash
# Create bucket
aws s3 mb s3://my-unique-bucket-name-12345

# Upload file
aws s3 cp myfile.txt s3://my-unique-bucket-name-12345/

# Upload directory
aws s3 sync ./website s3://my-unique-bucket-name-12345/ --acl public-read

# Download file
aws s3 cp s3://my-unique-bucket-name-12345/myfile.txt ./

# List objects
aws s3 ls s3://my-unique-bucket-name-12345/

# Delete object
aws s3 rm s3://my-unique-bucket-name-12345/myfile.txt

# Delete bucket
aws s3 rb s3://my-unique-bucket-name-12345 --force

# Make bucket website
aws s3 website s3://my-unique-bucket-name-12345/ \
  --index-document index.html \
  --error-document error.html
```

**S3 Storage Classes:**
```
Standard         - Frequent access ($0.023/GB/month)
Intelligent-Tier - Auto-move between tiers
Standard-IA      - Infrequent access ($0.0125/GB/month)
Glacier          - Archive ($0.004/GB/month)
Glacier Deep     - Long-term archive ($0.00099/GB/month)

Use case:
- Standard: Website assets, logs
- IA: Backups, older data
- Glacier: Compliance, archives
```

---

### RDS (Relational Database Service)

**Managed databases (PostgreSQL, MySQL, etc.)**

```bash
# Create PostgreSQL database
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 15.3 \
  --master-username admin \
  --master-user-password MySecurePass123! \
  --allocated-storage 100 \
  --backup-retention-period 7 \
  --multi-az \  # High availability
  --publicly-accessible

# Get endpoint
aws rds describe-db-instances \
  --db-instance-identifier mydb \
  --query 'DBInstances[0].Endpoint.Address' \
  --output text

# Connect
psql -h mydb.abc123.us-west-2.rds.amazonaws.com -U admin -d postgres

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-$(date +%Y%m%d)

# Delete database
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot
```

---

### Lambda (Serverless Functions)

**Run code without managing servers**

```python
# lambda_function.py
import json

def lambda_handler(event, context):
    name = event.get('name', 'World')
    return {
        'statusCode': 200,
        'body': json.dumps(f'Hello, {name}!')
    }
```

```bash
# Create Lambda function
zip function.zip lambda_function.py

aws lambda create-function \
  --function-name hello-world \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name hello-world \
  --payload '{"name": "DevOps"}' \
  response.json

cat response.json
# {"statusCode": 200, "body": "\"Hello, DevOps!\""}

# Update function
aws lambda update-function-code \
  --function-name hello-world \
  --zip-file fileb://function.zip

# Delete function
aws lambda delete-function --function-name hello-world
```

---

### CloudWatch (Monitoring & Logs)

**Monitoring, logging, alerting**

```bash
# View EC2 metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
  --start-time 2024-10-22T00:00:00Z \
  --end-time 2024-10-22T23:59:59Z \
  --period 3600 \
  --statistics Average

# Create alarm (CPU > 80%)
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "Alert when CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-west-2:123456789012:alerts

# View logs
aws logs tail /aws/lambda/hello-world --follow

# Create log group
aws logs create-log-group --log-group-name /my-app/logs

# Query logs (CloudWatch Insights)
aws logs start-query \
  --log-group-name /aws/lambda/hello-world \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc'
```

---

## Azure (Microsoft)

### 📖 Simple Explanation

**Azure = Microsoft's cloud**
- Started 2010
- 100+ services
- 23% market share (#2)
- Strong Windows/.NET integration
- Used by: BMW, Adobe, Samsung

---

### Azure VMs

**Virtual machines**

```bash
# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Create resource group
az group create --name myResourceGroup --location westus2

# Create VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# SSH into VM
ssh azureuser@<public-ip>

# Stop VM
az vm stop --resource-group myResourceGroup --name myVM

# Delete VM
az vm delete --resource-group myResourceGroup --name myVM --yes
```

---

### AKS (Azure Kubernetes Service)

**Managed Kubernetes**

```bash
# Create AKS cluster
az aks create \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_D2s_v3 \
  --enable-addons monitoring \
  --generate-ssh-keys

# Get credentials
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster

# Deploy application
kubectl apply -f deployment.yaml

# Scale cluster
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 5

# Upgrade Kubernetes version
az aks upgrade \
  --resource-group myResourceGroup \
  --name myAKSCluster \
  --kubernetes-version 1.28.0

# Delete cluster
az aks delete --resource-group myResourceGroup --name myAKSCluster --yes
```

---

### Azure Blob Storage

**Object storage (like S3)**

```bash
# Create storage account
az storage account create \
  --name mystorageaccount12345 \
  --resource-group myResourceGroup \
  --location westus2 \
  --sku Standard_LRS

# Create container
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount12345

# Upload file
az storage blob upload \
  --account-name mystorageaccount12345 \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt

# Download file
az storage blob download \
  --account-name mystorageaccount12345 \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./downloaded.txt

# List blobs
az storage blob list \
  --account-name mystorageaccount12345 \
  --container-name mycontainer \
  --output table
```

---

## GCP (Google Cloud Platform)

### 📖 Simple Explanation

**GCP = Google's cloud**
- Started 2008
- 100+ services
- 11% market share (#3)
- Best for: Big data, ML, Kubernetes (invented by Google)
- Used by: Spotify, Twitter, Snap

---

### Compute Engine

**Virtual machines**

```bash
# Install gcloud CLI
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
gcloud init

# Create VM
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB

# SSH into VM
gcloud compute ssh my-vm --zone=us-central1-a

# Stop VM
gcloud compute instances stop my-vm --zone=us-central1-a

# Delete VM
gcloud compute instances delete my-vm --zone=us-central1-a
```

---

### GKE (Google Kubernetes Engine)

**Managed Kubernetes (best-in-class)**

```bash
# Create GKE cluster
gcloud container clusters create my-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --machine-type n1-standard-2 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 10 \
  --enable-autorepair \
  --enable-autoupgrade

# Get credentials
gcloud container clusters get-credentials my-cluster --zone us-central1-a

# Deploy application
kubectl create deployment nginx --image=nginx:latest
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Scale cluster
gcloud container clusters resize my-cluster --num-nodes 5 --zone us-central1-a

# Delete cluster
gcloud container clusters delete my-cluster --zone us-central1-a
```

---

### Cloud Storage

**Object storage (like S3)**

```bash
# Create bucket
gsutil mb gs://my-unique-bucket-name-12345/

# Upload file
gsutil cp myfile.txt gs://my-unique-bucket-name-12345/

# Upload directory
gsutil -m cp -r ./website gs://my-unique-bucket-name-12345/

# Download file
gsutil cp gs://my-unique-bucket-name-12345/myfile.txt ./

# List objects
gsutil ls gs://my-unique-bucket-name-12345/

# Delete object
gsutil rm gs://my-unique-bucket-name-12345/myfile.txt

# Delete bucket
gsutil rb gs://my-unique-bucket-name-12345/

# Make bucket public
gsutil iam ch allUsers:objectViewer gs://my-unique-bucket-name-12345
```

---

## Cloud Networking

### 📖 Simple Explanation

**Cloud networking = Virtual data center**

**Key Concepts:**
- **VPC**: Virtual Private Cloud (your private network)
- **Subnet**: Segment of VPC (public, private)
- **Security Group**: Firewall rules
- **Load Balancer**: Distributes traffic
- **Internet Gateway**: Connect to internet

```
┌─────────────────────────────────────────┐
│            VPC (10.0.0.0/16)            │
│                                         │
│  ┌────────────────────────────────┐    │
│  │  Public Subnet (10.0.1.0/24)   │    │
│  │  ┌─────────┐    ┌─────────┐   │    │
│  │  │   Web   │    │   Web   │   │    │
│  │  │ Server  │    │ Server  │   │    │
│  │  └─────────┘    └─────────┘   │    │
│  └────────────────────────────────┘    │
│             ↕ Load Balancer             │
│  ┌────────────────────────────────┐    │
│  │  Private Subnet (10.0.2.0/24)  │    │
│  │  ┌──────────┐   ┌──────────┐  │    │
│  │  │ Database │   │ Database │  │    │
│  │  └──────────┘   └──────────┘  │    │
│  └────────────────────────────────┘    │
└─────────────────────────────────────────┘
         ↕ Internet Gateway
```

---

### AWS VPC

```bash
# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet (public)
aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-west-2a

# Create internet gateway
aws ec2 create-internet-gateway

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0123456789abcdef0 \
  --vpc-id vpc-0123456789abcdef0

# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-0123456789abcdef0

# Add inbound rule (allow HTTP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Add inbound rule (allow SSH)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

---

## IAM & Security

### 📖 Simple Explanation

**IAM = Who can do what**

**Key Concepts:**
- **User**: Person or service
- **Group**: Collection of users
- **Role**: Set of permissions (assumed)
- **Policy**: JSON document (what's allowed)

**Principle of Least Privilege:** Give only minimum permissions needed

---

### AWS IAM

```bash
# Create user
aws iam create-user --user-name developer

# Create group
aws iam create-group --group-name developers

# Add user to group
aws iam add-user-to-group --user-name developer --group-name developers

# Attach policy to group (S3 read access)
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create custom policy
cat > policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name EC2-Start-Stop \
  --policy-document file://policy.json

# Create role for EC2 instance
cat > trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

**IAM Best Practices:**
- ✅ Use MFA (Multi-Factor Authentication)
- ✅ Use roles instead of access keys
- ✅ Principle of least privilege
- ✅ Rotate credentials regularly
- ✅ Use AWS Organizations for multi-account
- ✅ Enable CloudTrail (audit logs)

---

## Storage Options

### 📖 Simple Explanation

**3 Types of Storage:**

**1. Object Storage (S3, Blob, Cloud Storage)**
```
Use for: Files, backups, static websites
Access: HTTP/HTTPS
Advantages: Unlimited size, cheap, durable
Disadvantages: Not mounted as filesystem
```

**2. Block Storage (EBS, Azure Disk, Persistent Disk)**
```
Use for: Database volumes, OS disks
Access: Attached to VM like hard drive
Advantages: Fast, can use filesystem
Disadvantages: Limited size, expensive
```

**3. File Storage (EFS, Azure Files, Filestore)**
```
Use for: Shared files across servers
Access: NFS/SMB protocols
Advantages: Multiple servers can access
Disadvantages: More expensive than object
```

---

### Comparison

| Feature | Object | Block | File |
|---------|--------|-------|------|
| **Use Case** | Static files, backups | Databases, VMs | Shared files |
| **Access** | HTTP API | Block device | NFS/SMB |
| **Performance** | Good | Excellent | Good |
| **Cost** | $0.023/GB/mo | $0.10/GB/mo | $0.30/GB/mo |
| **Multi-attach** | Yes (HTTP) | No (single VM) | Yes (NFS) |
| **Max Size** | Unlimited | 16 TB | Petabytes |

**Examples:**
- **Website images** → Object (S3)
- **PostgreSQL database** → Block (EBS)
- **Shared configs** → File (EFS)

---

## Managed Kubernetes

### Comparison: EKS vs AKS vs GKE

| Feature | AWS EKS | Azure AKS | GCP GKE |
|---------|---------|-----------|---------|
| **Cost** | $0.10/hr control plane | Free control plane | $0.10/hr control plane |
| **Auto-upgrade** | Manual | Automatic | Automatic |
| **Node pools** | Yes | Yes | Yes |
| **GPU support** | Yes | Yes | Yes |
| **Serverless** | Fargate | Virtual nodes | Autopilot |
| **Ease of use** | Complex | Easy | Easiest |
| **Best for** | AWS ecosystem | Azure/.NET apps | K8s features |

**Recommendation:**
- Already on AWS? → EKS
- Windows containers? → AKS
- Pure Kubernetes experience? → GKE
- Multi-cloud? → GKE (most portable)

---

## Serverless

### 📖 Simple Explanation

**Serverless = Run code without managing servers**

**Comparison:**

```
Traditional:
1. Provision EC2 instance ($50/month)
2. Runs 24/7
3. Pay even when idle
4. You manage OS, updates, scaling

Serverless:
1. Upload code
2. Runs only when triggered
3. Pay per execution ($0.0000002 per request)
4. Cloud manages everything
```

**When to use:**
- ✅ Event-driven (S3 upload triggers processing)
- ✅ Sporadic traffic (used few times/day)
- ✅ Short-running (<15 minutes)
- ❌ Long-running processes
- ❌ Constant traffic (more expensive)
- ❌ Need persistent connections

---

### Serverless Options

**AWS Lambda:**
```python
import json

def lambda_handler(event, context):
    # Triggered by S3 upload
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']
    print(f"File uploaded: {bucket}/{key}")
    # Process file...
    return {'statusCode': 200}
```

**Azure Functions:**
```python
import azure.functions as func

def main(req: func.HttpRequest) -> func.HttpResponse:
    name = req.params.get('name')
    return func.HttpResponse(f"Hello, {name}!")
```

**GCP Cloud Functions:**
```python
def hello_world(request):
    name = request.args.get('name', 'World')
    return f'Hello, {name}!'
```

---

## Cost Optimization

### 📖 Simple Explanation

**Cloud costs can explode if not managed!**

**Common Mistakes:**
- Forgot to turn off test servers ($5,000/month wasted)
- Over-provisioned instances (c5.4xlarge when t3.small enough)
- No auto-scaling (paying for 10 servers when need 2)
- No reserved instances/savings plans
- Expensive data transfer

---

### Cost Optimization Strategies

**1. Right-Sizing**
```bash
# AWS: Check underutilized instances
aws ce get-rightsizing-recommendation

# Use smaller instances
t3.medium instead of c5.xlarge = 70% savings
```

**2. Reserved Instances / Savings Plans**
```
On-Demand: $0.38/hour ($278/month)
Reserved (1 year): $0.23/hour ($168/month) → 40% savings
Reserved (3 years): $0.15/hour ($110/month) → 60% savings

Use for: Steady-state workloads (databases, always-on services)
```

**3. Spot Instances**
```
On-Demand: $0.38/hour
Spot: $0.11/hour → 70% savings!

Caveat: Can be terminated with 2-minute notice
Use for: Batch processing, stateless apps, CI/CD
```

**4. Auto-Scaling**
```
Manual: 10 instances 24/7 = $2,780/month
Auto-scale: 
- Business hours (8am-8pm): 10 instances
- Night (8pm-8am): 2 instances
Cost: ~$1,400/month → 50% savings
```

**5. S3 Lifecycle Policies**
```yaml
# Move old files to cheaper storage
lifecycle:
  - name: archive-old-data
    transition:
      - days: 30
        storage_class: STANDARD_IA  # -45% cost
      - days: 90
        storage_class: GLACIER       # -80% cost
    expiration:
      days: 365  # Delete after 1 year
```

**6. Monitor and Alert**
```bash
# AWS Cost Explorer
aws ce get-cost-and-usage \
  --time-period Start=2024-10-01,End=2024-10-31 \
  --granularity MONTHLY \
  --metrics BlendedCost

# Set budget alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

---

## Multi-Cloud Strategy

### 📖 Simple Explanation

**Multi-Cloud = Use multiple cloud providers**

**Why Multi-Cloud?**
- ✅ Avoid vendor lock-in
- ✅ Best-of-breed services (AWS ML, Azure AD, GCP BigQuery)
- ✅ Compliance (data must stay in specific regions)
- ✅ Disaster recovery (backup to different cloud)
- ✅ Negotiate better pricing

**Challenges:**
- ❌ Complexity (3x tools to learn)
- ❌ Higher costs (no volume discounts)
- ❌ More management overhead
- ❌ Data transfer costs

---

### Multi-Cloud Tools

**Kubernetes** (portable across clouds)
```yaml
# Same YAML works on EKS, AKS, GKE
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: myapp:latest
```

**Terraform** (provision any cloud)
```hcl
# AWS
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
}

# Azure
resource "azurerm_virtual_machine" "web" {
  name     = "web-vm"
  vm_size  = "Standard_B2s"
}

# GCP
resource "google_compute_instance" "web" {
  name         = "web-vm"
  machine_type = "e2-medium"
}
```

---

## Real-World Scenario

### Deploy Multi-Tier Application on AWS

**Architecture:**
- Load Balancer (ALB)
- Web servers (EC2 Auto Scaling Group)
- Application servers (EKS)
- Database (RDS PostgreSQL)
- Cache (ElastiCache Redis)
- Storage (S3)
- Monitoring (CloudWatch)

**Steps:**

```bash
# 1. Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# 2. Create subnets (public & private in 2 AZs)
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24 --availability-zone us-west-2a
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.2.0/24 --availability-zone us-west-2b

# 3. Create RDS database
aws rds create-db-instance \
  --db-instance-identifier prod-db \
  --engine postgres \
  --engine-version 15.3 \
  --db-instance-class db.r5.xlarge \
  --allocated-storage 500 \
  --multi-az \
  --backup-retention-period 30

# 4. Create ElastiCache Redis
aws elasticache create-cache-cluster \
  --cache-cluster-id prod-cache \
  --engine redis \
  --cache-node-type cache.r5.large \
  --num-cache-nodes 2

# 5. Create EKS cluster
eksctl create cluster \
  --name prod-cluster \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type m5.xlarge \
  --nodes 5 \
  --nodes-min 3 \
  --nodes-max 20 \
  --managed \
  --enable-ssm

# 6. Deploy application to EKS
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 10
  template:
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        env:
        - name: DATABASE_URL
          value: "postgres://prod-db.xxx.rds.amazonaws.com:5432/mydb"
        - name: REDIS_URL
          value: "redis://prod-cache.xxx.cache.amazonaws.com:6379"
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
---
apiVersion: v1
kind: Service
metadata:
  name: webapp
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: webapp
EOF

# 7. Configure auto-scaling
kubectl autoscale deployment webapp --cpu-percent=70 --min=10 --max=100

# 8. Setup CloudWatch monitoring
aws cloudwatch put-metric-alarm \
  --alarm-name high-error-rate \
  --metric-name ErrorCount \
  --namespace MyApp \
  --statistic Sum \
  --period 300 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold

# 9. Configure backups
aws backup create-backup-plan \
  --backup-plan file://backup-plan.json
```

---

## Chapter Summary

### What You Learned

**1. AWS Essentials**
- ✅ EC2 (VMs), EKS (Kubernetes), Lambda (Serverless)
- ✅ S3 (Object storage), RDS (Databases)
- ✅ CloudWatch (Monitoring)
- ✅ IAM (Security)

**2. Azure Essentials**
- ✅ VMs, AKS (Kubernetes), Functions
- ✅ Blob Storage, SQL Database
- ✅ Azure Monitor

**3. GCP Essentials**
- ✅ Compute Engine, GKE (best K8s)
- ✅ Cloud Storage, Cloud SQL
- ✅ Stackdriver

**4. Networking & Security**
- ✅ VPCs, Subnets, Security Groups
- ✅ Load Balancers
- ✅ IAM roles, policies, least privilege

**5. Storage Options**
- ✅ Object (S3), Block (EBS), File (EFS)
- ✅ When to use each

**6. Cost Optimization**
- ✅ Right-sizing, Reserved Instances, Spot
- ✅ Auto-scaling, lifecycle policies
- ✅ Monitoring and budgets

---

### 🎯 Interview Quick Tips

**Q1: AWS vs Azure vs GCP?**
```
AWS: Largest, most services, 32% market
Azure: Best for Microsoft shops, 23% market
GCP: Best for K8s/ML, 11% market

Choose based on:
- Existing skills/ecosystem
- Specific service needs
- Pricing
- Region availability
```

**Q2: When to use Serverless vs Containers vs VMs?**
```
Serverless: Event-driven, sporadic, < 15min
Containers: Microservices, portable, consistent
VMs: Legacy apps, specific OS needs

Most common: Containers (Kubernetes)
```

**Q3: How do you secure cloud resources?**
```
1. IAM: Least privilege, MFA, rotate keys
2. Network: VPC, security groups, private subnets
3. Encryption: At rest and in transit
4. Monitoring: CloudTrail, alerts
5. Compliance: CIS benchmarks
```

**Q4: How do you optimize cloud costs?**
```
1. Right-size instances (start small)
2. Reserved instances for steady workloads
3. Spot instances for batch jobs
4. Auto-scaling (scale down when idle)
5. S3 lifecycle (move to cheaper tiers)
6. Delete unused resources
7. Monitor and alert on spend
```

**Q5: Explain your cloud architecture**
```
Multi-tier, high-availability:
- Load Balancer (distribute traffic)
- Web tier (Auto Scaling Group, 2+ AZs)
- App tier (Kubernetes, auto-scale)
- Data tier (RDS Multi-AZ, read replicas)
- Cache (Redis/Memcached)
- Storage (S3 for static files)
- Monitoring (CloudWatch, Grafana)
```

---

### 💡 Best Practices

**Cloud Architecture Checklist:**
- ✅ Multi-AZ deployment (high availability)
- ✅ Auto-scaling (handle traffic spikes)
- ✅ Managed services (RDS over self-hosted DB)
- ✅ Infrastructure as Code (Terraform)
- ✅ Monitoring and alerting (CloudWatch)
- ✅ Backup and disaster recovery
- ✅ Security groups (least privilege)
- ✅ Encryption (at rest, in transit)
- ✅ Cost monitoring (budgets, alerts)
- ✅ Tagging (for cost allocation)

---

### 📚 What's Next?

Now that you know cloud platforms, you're ready for:
- **Infrastructure as Code** (Terraform to automate all this!)
- **CI/CD** (Deploy to cloud automatically)
- **Monitoring** (Prometheus + CloudWatch)
- **AI/LLM Infrastructure** (Deploy models on cloud)

**Practice Suggestions:**
1. Create free tier accounts (AWS, Azure, GCP)
2. Deploy a 3-tier app on each cloud
3. Practice AWS CLI commands daily
4. Build a multi-cloud K8s deployment
5. Optimize costs on personal projects

---

### 📝 Practice Exercises

1. **Deploy web app on EC2 with RDS backend**
2. **Create EKS cluster, deploy sample app**
3. **Set up S3 bucket with lifecycle policy**
4. **Configure auto-scaling for web servers**
5. **Create VPC with public/private subnets**
6. **Set up CloudWatch alarms for cost & metrics**
7. **Compare same deployment on AWS vs GCP**
8. **Implement multi-region disaster recovery**

---

## 🎉 **Foundation Complete!**

You've now mastered the complete infrastructure foundation:
- ✅ Linux (command line, scripting)
- ✅ Docker (containers)
- ✅ Kubernetes (orchestration)
- ✅ Cloud Platforms (AWS, Azure, GCP)

**This represents 80% of core DevOps skills!**

---

**Next Chapter:** [Infrastructure as Code (Terraform)](06-infrastructure-as-code.md)

**Foundation Mastery:** ✅ Complete! You're ready for automation!

