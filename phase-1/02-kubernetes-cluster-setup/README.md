# 🚀 Kubernetes Cluster Setup on AWS EC2

This guide provides a **complete, step-by-step process** to set up a production-ready **Kubernetes cluster on AWS EC2 (Ubuntu)** using `kubeadm`. 

**Assumes EC2 instances are already launched** - focuses on Docker, Kubernetes components, CNI networking, and Ingress setup.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-blueviolet?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-20.10-green?logo=docker&logoColor=white)](https://www.docker.com/)
[![Calico](https://img.shields.io/badge/CNI-Calico-orange?logo=projectcalico&logoColor=white)](https://projectcalico.org/)

---

## 🧭 Architecture Overview

AWS VPC
├── Control Plane (Master Node)
│ └── kube-apiserver, controller-manager, scheduler
├── Worker Node 1
├── Worker Node 2
└── Pod Network (Calico CNI)

text

**Cluster Specs:**
- **Master Node**: 1 instance (t3.medium recommended)
- **Worker Nodes**: 1+ instances (t3.small/medium)
- **Pod CIDR**: `192.168.0.0/16`
- **CNI**: Calico
- **Ingress**: NGINX Controller

---

## 📋 Prerequisites

Before starting, ensure:

- **Ubuntu 20.04/22.04** EC2 instances launched
- **1 Master + 1+ Worker nodes** with same VPC
- **Security Group** allows:
SSH: TCP 22
K8s API Server: TCP 6443
NodePort Range: TCP 30000-32767
Intra-cluster: All traffic (same SG)

text

- **`sudo` access** on all nodes
- **Private IP connectivity** between all nodes

---

## 🚀 Step-by-Step Setup

### 🐳 Step 1: Install Docker
**Run on ALL nodes (Master + Workers)**

```bash
sudo apt update && sudo apt install docker.io -y
sudo systemctl enable docker && sudo systemctl start docker
sudo chmod 666 /var/run/docker.sock
Verify:

bash
docker --version
☸️ Step 2: Install Kubernetes Components
Run on ALL nodes

bash
# Add Kubernetes apt repository
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install specific versions
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl
Verify:

bash
kubeadm version
kubectl version --client
🔧 Step 3: Configure Kernel & Sysctl
Run on ALL nodes (REQUIRED)

bash
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
🎯 Step 4: Initialize Master Node
Run ONLY on Master node

bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
Configure kubectl for current user:

bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
💾 SAVE THE kubeadm join command printed at the end!

🔗 Step 5: Join Worker Nodes
Run on EACH Worker node

If previously joined:

bash
sudo kubeadm reset -f
Join cluster:

bash
sudo kubeadm join <MASTER_PRIVATE_IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
🌐 Step 6: Install Calico CNI
Run on Master node

bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
Wait 1-2 minutes for pods to be ready:

bash
kubectl get pods -n calico-system
🚪 Step 7: Install NGINX Ingress Controller
Run on Master node

bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
✅ Step 8: Verify Cluster
Run on Master node

bash
# Check nodes status
kubectl get nodes

# Check all pods
kubectl get pods --all-namespaces

# Check ingress controller
kubectl get pods -n ingress-nginx
Expected Output:

text
NAME               STATUS   ROLES           AGE    VERSION
master-node        Ready    control-plane   5m     v1.28.1
worker-1           Ready    <none>          2m     v1.28.1
worker-2           Ready    <none>          1m     v1.28.1