# 🚀 Kubeadm Installation Guide

This guide provides a comprehensive, step-by-step walkthrough for setting up a production-ready Kubernetes cluster using `kubeadm` on Ubuntu.

---

## 📋 Prerequisites

| Requirement | Specification |
| :--- | :--- |
| **Operating System** | Ubuntu OS (Xenial or later) |
| **Instance Type** | `t2.medium` or higher (minimum 2 vCPUs, 4GB RAM) |
| **Nodes** | Minimum 2 instances (1 Master, 1 Worker) |

---

## ☁️ AWS Setup Requirements

Before you begin, ensure your AWS environment is correctly configured:

1.  **Shared Security Group**: All instances must belong to the same **Security Group**.
2.  **Inbound Rules**:
    *   🔓 **Port 6443**: Allow inbound traffic from worker nodes (Kubernetes API).
    *   🔓 **Port 22**: Allow SSH access for cluster management.
    *   🔓 **Port 30080**: Allow traffic for your NodePort services (added later).

> [!IMPORTANT]
> **Step 1: Select the Security Group During Instance Creation**
> Make sure to apply the security group mentioned above when launching your EC2 instances.

---

## 🛠️ Phase 1: Preparation (Execute on BOTH Nodes)

Run these steps on both the **Master** and any **Worker** nodes.

### 1. Update and Upgrade System
Keep your system packages up to date:
```bash
sudo apt update
sudo apt upgrade -y
```

### 2. Disable Swap
Kubernetes requires swap to be disabled to function correctly:
```bash
sudo swapoff -a
```

### 3. Load Kernel Modules
Enable necessary modules for Kubernetes networking:
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### 4. Configure Sysctl Parameters
Optimize networking settings for bridge traffic:
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
lsmod | grep br_netfilter
lsmod | grep overlay
```

### 5. Install Containerd Runtime
Set up the container runtime environment:
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo \"$VERSION_CODENAME\") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y containerd.io

containerd config default | sed -e 's/SystemdCgroup = false/SystemdCgroup = true/' -e 's/sandbox_image = "registry.k8s.io\/pause:3.6"/sandbox_image = "registry.k8s.io\/pause:3.9"/' | sudo tee /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
containerd --version
```

### 6. Install Kubernetes Components
Install the core binaries for the cluster:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 👑 Phase 2: Cluster Initiation (Master Node ONLY)

Perform these steps **only** on the Master node.

### 1. Initialize the Control Plane
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 
```

### 2. Configure Local Kubeconfig
```bash
mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

### 3. Install Calico Network Plugin
Choose one of the following commands to install the network overlay:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml --validate=false
# OR
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml --validate=false
```

### 4. Generate Join Command
Get the command to connect your worker nodes:
```bash
kubeadm token create --print-join-command
```

> [!TIP]
> **Copy the generated output!** You will need this to join your worker nodes to the cluster in the next phase.

---

## 👷 Phase 3: Joining Workers (Worker Nodes ALL)

Execute these steps on all your Worker nodes.

### 1. Pre-flight Checks
Reset any existing configuration to ensure a clean slate:
```bash
sudo kubeadm reset pre-flight checks
```

### 2. Join the Cluster
Paste the command you generated on the master node, appending the CRI socket flag:
```bash
sudo <copy-and-paste-kubeadm-join-from-master> --cri-socket "unix:///run/containerd/containerd.sock" --v=5
```

---

## 📦 Phase 4: Deploying Applications (Master Node)

Once the cluster is healthy, deploy your first application.

### 1. Clone the repository:
```bash
git clone https://github.com/NI-Shourav/k8sStarter.git
cd k8sStarter
cd kubeadm
```

### 2. Apply manifests:
```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl get pods -n kubernetes-cluster
kubectl apply -f services.yaml
kubectl get svc -n kubernetes-cluster
```

---

## 🔍 Verification & Maintenance

### Check Container Status (Worker Node)
```bash
sudo crictl ps -a
```

### Test in Browser
Access your application via the worker node's IP:
```bash
http://<worker-node-ip>:30080
```

### Delete a Pod (Testing Self-Healing)
```bash
kubectl delete pod <pod_id> -n kubernetes-cluster
```

---

✨ **Cheers! Your cluster is ready.**
