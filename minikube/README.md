# 🚀 Minikube: Installation & Nginx Deployment Guide

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white) 
![Ubuntu](https://img.shields.io/badge/Ubuntu-E94333?style=for-the-badge&logo=ubuntu&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)

Welcome! This guide provides a streamlined approach to installing **Minikube** on Ubuntu and deploying your first **Nginx** application. Whether you're working locally or on a cloud instance, this will help you get a single-node Kubernetes cluster running with a live app in minutes.

---

## 📋 Table of Contents
1. [Pre-requisites](#-pre-requisites)
2. [Step 1: Update System Packages](#step-1-update-system-packages)
3. [Step 2: Install Required Packages](#step-2-install-required-packages)
4. [Step 3: Install Docker](#step-3-install-docker)
5. [Step 4: Install Minikube](#step-4-install-minikube)
6. [Step 5: Install kubectl](#step-5-install-kubectl)
7. [Step 6: Start Minikube](#step-6-start-minikube)
8. [Step 7: Check Cluster Status](#step-7-verify-cluster-status-)
9. [Step 8: Deploy First App](#step-8-deploy-your-first-application-)
10. [Step 9: Interactive Exploration](#step-9-interactive-exploration-docker--node-)
11. [Step 10: Stop & Cleanup](#step-10-stop--delete-cluster-)
12. [Step 11: Uninstallation](#step-11-complete-uninstallation-optional-)

---

## 🛠️ Pre-requisites

Before starting, ensure your system meets these minimum hardware requirements:

*   **CPU:** 2 cores or more
*   **Memory:** 2GB+ of free RAM
*   **Disk:** 20GB+ of free space
*   **Privileges:** `sudo` access

---

## 🏗️ Step-by-Step Installation

### Step 1: Update System Packages
Keep your system fresh and ready.
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Essential Dependencies
Grab the necessary tools for the installation process.
```bash
sudo apt install -y curl wget apt-transport-https
```

---

### Step 3: Install Docker 🐳
Minikube needs a driver to run. Docker is the most popular and efficient choice for local development.

1. **Install Docker Engine:**
   ```bash
   sudo apt install -y docker.io
   ```

2. **Enable Service:**
   ```bash
   sudo systemctl enable --now docker
   ```

3. **Configure Permissions:**
   Add your user to the `docker` group to run commands without `sudo`.
   ```bash
   sudo usermod -aG docker $USER
   ```
   > [!IMPORTANT]
   > You must **log out and log back in** for group changes to take effect.
   > Or run: `newgrp docker`

   *Quick fix if you're in a hurry:*
   ```bash
   sudo chown $USER /var/run/docker.sock
   ```

---

### Step 4: Install Minikube 📦
Now, let's fetch the actual Minikube binary.

```bash
# Download the latest binary
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64

# Install to /usr/local/bin
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

**Verify the installation:**
```bash
minikube version
```

---

### Step 5: Install kubectl 🔧
`kubectl` is the command-line tool used to interact with your Kubernetes cluster.

1. **Download:**
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   ```

2. **Install:**
   ```bash
   chmod +x kubectl
   sudo mv kubectl /usr/local/bin/
   ```

---

### Step 6: Start Your Cluster! 🚀
Now, start Minikube with the following command:

```bash
minikube start --driver=docker --vm=true
```

If you already have Docker installed, start Minikube with the following command:

```bash
minikube start
```

This will start a single-node Kubernetes cluster inside a Docker container and your Kubernetes cluster should be ready.

---

### Step 7: Verify Cluster Status ✅
Ensure the node is ready.
```bash
# Check Minikube status
minikube status

# Check nodes with kubectl
kubectl get nodes
```

---

### Step 8: Deploy Your First Application 🌎
Let's run a simple Nginx server to test the cluster.

1. **Run the Nginx Pod:**
   Replace `<pod-name>` with your desired name.
   ```bash
   kubectl run <pod-name> --image=nginx:latest
   ```

2. **Check Pod Status:**
   ```bash
   kubectl get pods
   ```

3. **Get More Details (IPs and Nodes):**
   ```bash
   kubectl get pods -o wide
   ```

---

### Step 9: Interactive Exploration (Docker & Node) 🔍
Since Minikube is running with the Docker driver, the Kubernetes "node" is actually a Docker container on your host.

1. **Find the Minikube Container:**
   ```bash
   docker ps
   ```
   *Look for a container named `minikube`.*

2. **Access the Node's Terminal:**
   Replace `<container_id>` with the ID found in the previous step.
   ```bash
   docker exec -it <container_id> bash
   ```
   *Inside here, you are "inside" the Kubernetes node. Type `exit` to return to your host.*

---

### Step 10: Stop & Delete Cluster 🧹
When you're done, you can stop the cluster to save resources or delete it to start fresh.

**To pause the cluster:**
```bash
minikube stop
```

**To delete the cluster data:**
```bash
minikube delete
```

---

### Step 11: Complete Uninstallation (Optional) 🚫
To completely remove Minikube and its tools from your system:

1. **Navigate to the binary directory:**
   ```bash
   cd /usr/local/bin/
   ```

2. **Remove the binary:**
   ```bash
   sudo rm -rf minikube
   ```

3. **Return home:**
   ```bash
   cd ~/
   ```

---

## ✨ Happy K8s-ing!

> Use `minikube dashboard` to open a web-browser based UI for your cluster!

---
*Created with ❤️ for the Kubernetes community.*
