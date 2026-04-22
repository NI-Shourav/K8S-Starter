# ☸️ K8S-Starter

Welcome to **K8S-Starter**, a comprehensive resource for setting up and managing Kubernetes clusters. This repository provides automated scripts and documentation for **Kind**, **Minikube**, **Kubeadm**, and **AWS EKS** environments.

## 📂 Project Structure

This repository is organized into the following sections:

- **[kind/](./kind/)**: Automated setup for Kubernetes-in-Docker (Kind).
  - Pre-configured cluster settings.
  - Automated installation script for Docker, Kind, and kubectl.
  - Hands-on Kubernetes manifests (Pods, Deployments, Namespaces).
- **[minikube/](./minikube/)**: Step-by-step guide for Minikube installation on Ubuntu.
  - Resource optimization tips.
  - First application deployment walkthrough.
- **[kubeadm/](./kubeadm/)**: Production-ready multi-node cluster setup.
  - Comprehensive guide for Ubuntu/AWS environments.
  - Master and Worker node configuration.
  - Integrated networking with Calico.
- **[eks/](./eks/)**: Managed Kubernetes Service on AWS.
  - Step-by-step EKS cluster provisioning using `eksctl`.
  - IAM and AWS CLI configuration.
  - Full-stack application deployment with Load Balancers.

## 🚀 Getting Started

### Clone the Repository

```bash
git clone https://github.com/NI-Shourav/K8S-Starter.git
cd K8S-Starter
```

### Choose Your Path

- If you want a lightweight, Docker-based cluster, head over to the **[Kind Setup Guide](./kind/README.md)**.
- If you prefer a full-featured single-node cluster (ideal for Ubuntu), follow the **[Minikube Installation Guide](./minikube/minikube_installation.md)**.
- For a production-ready multi-node cluster, follow the **[Kubeadm Installation Guide](./kubeadm/README.md)**.
- For a managed, production-grade cloud cluster, follow the **[AWS EKS Setup Guide](./eks/README.md)**.

## 🛠️ Features

- ✅ **Automated Scripts:** No more manual binary downloads.
- ✅ **Best Practices:** Pre-configured YAML manifests for common resources.
- ✅ **Simplified Workflow:** Detailed, easy-to-follow documentation for beginners.

---

<div align="center">
  <sub>Developed with ❤️ for the Kubernetes community.</sub>
</div>
