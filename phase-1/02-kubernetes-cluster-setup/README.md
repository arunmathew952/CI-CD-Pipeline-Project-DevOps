# 🚀 Kubernetes Cluster Setup on AWS EC2

This guide documents the complete process of setting up a **Kubernetes cluster on AWS EC2 (Ubuntu)** using `kubeadm`.  
It assumes that **EC2 instances are already launched** and focuses on installing Docker, Kubernetes components, networking, and ingress.

---

## 🧭 Architecture Overview

AWS VPC
├── Control Plane (Master Node)
│ └── kube-apiserver, controller-manager, scheduler
├── Worker Node 1
├── Worker Node 2
└── Pod Network (Calico CNI)

yaml
Copy code

---

## 📋 Prerequisites

- Ubuntu 20.04 / 22.04 EC2 instances
- 1 Master node + 1 or more Worker nodes
- Security Group rules:
  - SSH → `22`
  - Kubernetes API → `6443`
  - NodePort → `30000–32767`
  - All traffic allowed between cluster nodes
- `sudo` access on all machines

---

## 🐳 Step 1: Install Docker  
**(Run on Master & Worker nodes)**

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo chmod 666 /var/run/docker.sock
Verify:

bash
Copy code
docker --version
☸️ Step 2: Install Kubernetes Components
(Run on Master & Worker nodes)

bash
Copy code
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key \
 | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl
Verify:

bash
Copy code
kubeadm version
kubectl version --client
🔧 Step 3: Kernel & Sysctl Configuration
(Mandatory – Run on all nodes)

bash
Copy code
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
✅ These settings persist across reboots.

🎯 Step 4: Initialize the Master Node
(Run only on Master node)

bash
Copy code
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
Configure kubectl
For normal user

bash
Copy code
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
For root user

bash
Copy code
export KUBECONFIG=/etc/kubernetes/admin.conf
📌 Save the kubeadm join command printed at the end.

🔗 Step 5: Join Worker Nodes
(Run on each Worker node)

If the node was previously joined:

bash
Copy code
sudo kubeadm reset -f
Join the cluster:

bash
Copy code
sudo kubeadm join <MASTER_PRIVATE_IP>:6443 \
 --token <TOKEN> \
 --discovery-token-ca-cert-hash sha256:<HASH>
🌐 Step 6: Install Pod Network (Calico)
bash
Copy code
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
🚪 Step 7: Install NGINX Ingress Controller
bash
Copy code
kubectl apply -f \
https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
✅ Step 8: Verify Cluster Status
bash
Copy code
kubectl get nodes
