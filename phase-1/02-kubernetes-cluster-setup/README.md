Kubernetes Cluster Setup on AWS EC2

This documentation covers setting up a Kubernetes cluster on Ubuntu EC2 instances (Master + Worker nodes) after EC2 instances are already launched. It includes steps for Docker, Kubernetes installation, networking (Calico), and Ingress controller (NGINX).

Table of Contents

Prerequisites

Install Docker

Install Kubernetes Components

Configure Kernel Modules & Sysctl

Initialize Master Node

Join Worker Nodes

Deploy Network & Ingress

Verify Cluster

Scripts

Screenshots

Prerequisites

Ubuntu 20.04 EC2 instances

Security groups allow:

SSH (22)

Kubernetes API (6443)

NodePorts (30000-32767)

Internal traffic between nodes (all TCP/UDP)

Key pair for SSH access

Instances reachable via private/public IPs

Install Docker

Run on Master and Worker nodes:

sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo chmod 666 /var/run/docker.sock


Verify Docker:

docker --version

Install Kubernetes Components

Run on Master and Worker nodes:

sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl


Verify installation:

kubeadm version
kubectl version --client

Configure Kernel Modules & Sysctl

Run on all nodes:

sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

Initialize Master Node

Run on Master node only:

sudo kubeadm init --pod-network-cidr=192.168.0.0/16


Set up kubectl access:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


Alternatively, for root user:
export KUBECONFIG=/etc/kubernetes/admin.conf


Copy the kubeadm join command output for the worker nodes—it will be used in the next step.
On each Worker node:

sudo kubeadm reset -f  # Only if previous attempts failed
sudo kubeadm join <MASTER_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>


Verify nodes on Master:

kubectl get nodes


If nodes show NotReady, wait a few minutes for network pods to start.

Deploy Network & Ingress
Deploy Calico CNI:
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Deploy NGINX Ingress Controller:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml

Verify Cluster

Check node status:

kubectl get nodes


Check system pods:

kubectl get pods -n kube-system -o wide

Scripts

You can create a shell script (k8s-setup.sh) with all repetitive steps:

#!/bin/bash
set -e

# Docker & Kubernetes prerequisites
sudo apt update
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock

sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl

# Kernel modules & sysctl
sudo modprobe br_netfilter
sudo sysctl --system
Join Worker Nodes


