<div align="center">
  <h1>KIND Cluster Setup Guide</h1>
  <p>🚀 <b>Automated Kubernetes-in-Docker environment setup</b></p>

  [![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
  [![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
  [![Kind](https://img.shields.io/badge/kind-v0.29.0-orange?style=for-the-badge&logo=kubernetes)](https://kind.sigs.k8s.io/)
</div>

---

## 📖 Overview

This repository provides an automated script and configuration files to set up a local Kubernetes cluster using **Kind (Kubernetes in Docker)**. It includes everything you need to go from zero to a fully functional cluster.

### 🛠 Prerequisites

Before you begin, ensure your machine meets the minimum requirements:

- **CPU:** 2+ Cores
- **Memory:** 4 GB+ RAM
- **Disk:** 10 GB+ Space

---

## ⚡ Quick Start

### 0️⃣ Clone the Repository
If you are setting this up on an EC2 instance, first clone the repository and navigate to the `kind` directory:

```bash
git clone https://github.com/<your-username>/kubernetes-starter.git
cd kubernetes-starter/kind
```

### 1️⃣ Install Dependencies
Run the provided installer script to set up Docker, Kind, and kubectl:

```bash
chmod +x installer.sh
./installer.sh
```

> [!NOTE]
> This script will detect your architecture (x86_64 or ARM64) and install the appropriate binaries.

### 2️⃣ Create Your Cluster
Spin up the cluster using the pre-configured settings:

```bash
kind create cluster --config=kind-config.yml --name=nur-cluster
```

> [!TIP]
> **Optional: Explore Kubernetes Config**
> You can manually verify the cluster configuration if needed:
> ```bash
> cd ~/.kube
> ls
> cat config
> ```

### 3️⃣ Verify the Setup
Check if your nodes are ready and the cluster is reachable:

```bash
kubectl get nodes
kubectl cluster-info --context kind-nur-cluster
```

### 4️⃣ Working with Manifests
Explore basic Kubernetes resource management using the provided YAML files:

```bash
# 1. Check existing namespaces
kubectl get namespace

# 2. Navigate to manifests directory
cd manifests
ls

# 3. Create a new namespace
kubectl apply -f namespace.yml
kubectl get namespace

# 4. Deploy and check a Pod
kubectl apply -f pod.yml
kubectl get pods -n nginx

# 5. Delete the temporary pod
kubectl delete pod nginx-pod -n nginx

# 6. Deploy a Deployment
kubectl apply -f deployment.yml
kubectl get pods -n nginx
kubectl get pods -n nginx -o wide

# 7. Scale the Deployment
# Scale down to 1 replica
kubectl scale deployment/nginx-deployment -n nginx --replicas=1
kubectl get pods -n nginx -o wide

# Scale up to 5 replicas
kubectl scale deployment/nginx-deployment -n nginx --replicas=5
kubectl get pods -n nginx -o wide
```

---

## 🗑 Cleanup
To delete the cluster and free up resources:

```bash
kind delete cluster --name nur-cluster
```

---

## 💡 Important Notes

- **Persistence:** Kind clusters are ephemeral. If Docker restarts, you may need to recreate the cluster.
- **Customization:** Update `kind-config.yml` to change node images or port mappings.
- **Multiple Clusters:** Support multiple isolated environments by using unique `--name` flags.

---

<div align="center">
  <sub>Developed with ❤️ for the Kubernetes community.</sub>
</div>
