# 🚀 Elastic Kubernetes Service (EKS) on AWS
## Complete Step-by-Step Deployment Guide

---

## 📋 Overview

**AWS EKS** is Amazon's managed Kubernetes service that handles auto-scaling and auto-healing, eliminating the operational burden of managing control planes yourself. This guide walks you through provisioning a full-stack application on EKS using `eksctl`.

### Key Benefits
- **Managed Control Plane**: AWS handles auto-scaling and auto-healing
- **Simplified Deployment**: Using `eksctl` reduces complexity
- **Serverless Option**: AWS Fargate allows you to run containers without managing EC2 instances
- **Infrastructure as Code**: Leverage Terraform or CloudFormation for reproducible deployments

---

## 📋 Prerequisites

Before starting, ensure you have:

- ✅ AWS Account with appropriate permissions
- ✅ Basic AWS knowledge (IAM, EC2, VPC)
- ✅ Terminal/SSH access
- ⚠️ **Region Selection**: Avoid N. Virginia or Ohio for cost efficiency

---

## 🛠️ Quick Start: 3 Ways to Create an EKS Cluster

1. **AWS Console** — UI-based approach (manual steps)
2. **eksctl** — Command-line tool (recommended for beginners)
3. **Terraform** — Infrastructure as Code (best for production)

This guide focuses on **Method 2: eksctl** ⚡

---

## 📍 Step-by-Step Guide

### **STEP 1: Create an EC2 Instance**

Launch a t2.micro instance. This machine is **not** part of your cluster—it's only used to trigger EKS cluster creation.

```bash
# Instance Type: t2.micro is sufficient
# This is your "control machine" for running eksctl commands
```

---

### **STEP 2: Connect via SSH & Update System**

```bash
ssh -i /path/to/key.pem ec2-user@<your-ec2-public-ip>

# Update system packages
sudo apt update
sudo apt upgrade -y
```

---

### **STEP 3: Create IAM User with Administrator Access**

Create a dedicated IAM user for EKS operations:

```
AWS Console → IAM → Users → Create User
  • Username: eks-user
  • Attach Policy Directly → AdministratorAccess
  • Create User
```

---

### **STEP 4: Install AWS CLI**

Download and install the AWS Command Line Interface on your EC2 instance:

```bash
# Download AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Install unzip utility
sudo apt install unzip -y

# Extract and install
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version

# Clean up
rm -rf awscliv2.zip aws/
```

---

### **STEP 5: Configure AWS Credentials**

Link your IAM user credentials to the EC2 machine:

```
AWS Console → IAM → Users → eks-user
  • Create Access Key → CLI → Next → Create Access Key
  • Copy Access Key ID and Secret Access Key
```

Then configure on your EC2 instance:

```bash
aws configure
# When prompted:
# AWS Access Key ID: [paste your access key]
# AWS Secret Access Key: [paste your secret key]
# Default region: us-west-2 (or your preferred region)
# Default output format: [leave as None unless you prefer json]
```

---

### **STEP 6A: Install eksctl**

`eksctl` is the command-line tool to create and manage EKS clusters:

```bash
# Download and install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

# Move to PATH
sudo mv /tmp/eksctl /usr/local/bin

# Verify installation
eksctl version
eksctl help
```

✅ **Note**: `eksctl` is the tool used to **create** an EKS cluster.

---

### **STEP 6B: Install kubectl**

`kubectl` is the Kubernetes CLI used to manage and interact with your cluster:

```bash
# Download kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Download SHA256 checksum for verification
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Verify integrity (should output: kubectl: OK)
echo "$(cat kubectl.sha256) kubectl" | sha256sum --check

# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client
```

✅ **Note**: `kubectl` is the tool used to **interact** with a Kubernetes cluster.

---

### **STEP 6C: Create the EKS Cluster** 🎯

Now the exciting part—launch your cluster!

#### Option A: Using c7i-flex.large (Recommended for Production)
```bash
eksctl create cluster \
  --name full-stack-cluster \
  --region us-west-2 \
  --node-type c7i-flex.large \
  --nodes-min 2 \
  --nodes-max 2
```

#### Option B: Using t2.medium (Cost-effective for Testing)
```bash
eksctl create cluster \
  --name full-stack-cluster \
  --region us-west-2 \
  --node-type t2.medium \
  --nodes-min 2 \
  --nodes-max 2
```

#### Option C: Using t3.micro (Minimal Testing)
```bash
eksctl create cluster \
  --name full-stack-cluster \
  --region us-west-2 \
  --node-type t3.micro \
  --nodes-min 2 \
  --nodes-max 2
```

**What happens next?**
- `eksctl` creates a VPC and provisions the Control Plane
- AWS CloudFormation allocates all necessary resources
- Worker nodes are launched and registered with the cluster
- ⏱️ **Patience**: This typically takes 10-15 minutes

---

### **STEP 7: Verify Cluster Creation**

```bash
# List all pods
kubectl get pods
# Note: This will show "No resources found in default namespace." because no pods are created yet.

# List all namespaces
kubectl get namespace

# List system pods
kubectl get pods -n kube-system

# List all worker nodes
kubectl get nodes

# List EKS clusters
eksctl get cluster
```

---

### **STEP 8: Clone Application Repository** (Optional)

```bash
git clone https://github.com/NI-Shourav/K8S-Starter.git
cd K8S-Starter/eks
```

---

### **STEP 9: Deploy Manifest Files**

Apply your Kubernetes YAML manifests to deploy applications:

```bash
# Deploy in correct order
kubectl apply -f namespace.yaml
kubectl apply -f mysql-secrets.yml
kubectl apply -f mysql-deployment.yml
kubectl apply -f mysql-svc.yml
kubectl apply -f two-tier-app-deployment.yml
kubectl apply -f two-tier-app-svc.yml
```

#### Get External IP (Access Your Application)
```bash
# Find the LoadBalancer external IP
kubectl get svc -n two-tier-ns

# Copy the EXTERNAL-IP and paste into your browser. This will take some time (approx 2 min) to live.
# Your application is now live! 🎉
```

---

### **STEP 10: Manage & Debug Pods**

#### View All Resources
```bash
kubectl get all -n two-tier-ns
```

#### Delete All Deployments
```bash
kubectl delete -f .
```

#### Debug Specific Pod
```bash
kubectl describe pod/mysql-6d576469c4-67929 -n two-tier-ns
```

#### View Pod Logs
```bash
kubectl logs mysql-98596f974-5crh5 -n two-tier-ns
```

#### ⚠️ Handle Scheduling Errors
If you see: `Warning FailedScheduling — 0/2 nodes are available: 2 Too many pods`
- Your nodes have reached pod capacity
- Solutions:
  - Scale up node count: `eksctl scale nodegroup --cluster=<name> --nodes=3`
  - Use larger instance types
  - Implement pod resource limits

---

## 🧹 Cleanup & Cost Optimization

### Delete the EKS Cluster

```bash
eksctl delete cluster \
  --name full-stack-cluster \
  --region us-west-2
```

### Optional Manual Cleanup Checklist

After deletion, verify these resources are cleaned up:

#### 1. **CloudFormation Stacks**
```
AWS Console → CloudFormation → Delete any remaining stacks
(eksctl uses CloudFormation under the hood)
```

#### 2. **VPCs**
```
AWS Console → VPC Dashboard → Verify created VPC is removed
```

#### 3. **IAM Roles**
```
AWS Console → IAM → Roles → Delete cluster-specific roles
(Only if certain nothing depends on them)
```

#### 4. **EBS Volumes & Load Balancers**
```
AWS Console → EC2 → Elastic Block Store → Volumes
AWS Console → EC2 → Load Balancing → Load Balancers
Check for leftover resources and delete manually
```

#### 5. **Manually Created Resources**
⚠️ **These are NOT auto-deleted**:
- RDS databases
- S3 buckets
- Custom IAM roles
- Custom VPCs
- Any other AWS services created outside eksctl

---

## 🔄 Switching Between Multiple Clusters

If you have multiple EKS clusters, point `kubectl` to the correct one:

```bash
# Update kubeconfig for a specific cluster
aws eks update-kubeconfig \
  --name full-stack-cluster \
  --region us-west-2

# Verify you're connected to the right cluster
kubectl get nodes
```

---

## 💡 Pro Tips

| Tip | Details |
|-----|---------|
| **Regional Selection** | Avoid N. Virginia (us-east-1) and Ohio (us-east-2) for better pricing |
| **Node Types** | Use `t2.medium` for testing, `c7i-flex.large` for production workloads |
| **Auto-scaling** | Start with `--nodes-min 2 --nodes-max 2` and adjust based on load |
| **Cost Monitoring** | Use AWS Cost Explorer to track EKS cluster expenses |
| **Backup Control** | Enable cluster logging for audit trails: `eksctl enable logging --cluster=<name>` |

---

## 📚 Key Concepts

### Control Plane vs Data Plane
- **Control Plane** (AWS-Managed): Scheduler, API Server, etcd, Controller Manager
- **Data Plane** (Your Nodes): EC2 instances running your application pods

### eksctl vs kubectl
- **eksctl**: Creates and manages EKS clusters (infrastructure)
- **kubectl**: Manages Kubernetes resources inside clusters (applications)

### AWS Fargate (Alternative to EC2)
Serverless compute option that eliminates the need to manage EC2 instances:
```bash
# Create cluster with Fargate instead of EC2 nodes
eksctl create cluster --name serverless-cluster --fargate
```

---

## 🐛 Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| **Scheduling Errors** | Scale nodes: `eksctl scale nodegroup --nodes=3` |
| **Pod Pending** | Check resource requests: `kubectl describe pod <name>` |
| **Connection Timeout** | Verify security groups allow kubectl traffic |
| **Cluster Not Accessible** | Update kubeconfig: `aws eks update-kubeconfig --name <cluster>` |

---

## 🔗 Useful Resources

- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [eksctl Official Guide](https://eksctl.io/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

---

## 📝 Summary

You now have a complete, production-ready EKS cluster! Here's what you've accomplished:

✅ Provisioned managed Kubernetes infrastructure
✅ Installed and configured `eksctl` and `kubectl`
✅ Created an EKS cluster with worker nodes
✅ Deployed a full-stack application (optional)
✅ Learned cluster management and debugging

**Next Steps:**
- Deploy your own applications
- Implement auto-scaling policies
- Set up monitoring with CloudWatch
- Configure CI/CD pipelines with GitHub Actions

---

**Made with ❤️ for Kubernetes enthusiasts**

