🛠️ Section A: Prepare All Nodes
Perform these steps on both the Master and all Worker nodes.

Step 1: Disable Swap
Kubernetes requires swap to be off to manage resources accurately.

Bash

sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
Success Check: Run free -m. The "Swap" row should show 0.

Step 2: Load Kernel Modules
We need to enable the overlay and br_netfilter modules so the nodes can handle network traffic correctly.

Bash

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
Step 3: Configure Networking
This allows the bridge to see bridged traffic and enables IP forwarding.

Bash

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
Step 4: Install & Configure containerd
Containerd is our runtime (the engine that runs the containers).

Bash

sudo apt update && sudo apt install -y containerd

# Initialize default configuration
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# Set SystemdCgroup to true (required for Kubelet stability)
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
Step 5: Install Kubernetes Tools
We install the "Big Three": kubeadm (the installer), kubelet (the node manager), and kubectl (the command tool).

Bash

sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gpg
sudo mkdir -p /etc/apt/keyrings

# Add the official Kubernetes keyring
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install specific version 1.28
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl
🧠 Section B: Initialize the Master
Perform these steps ONLY on the Master node.

Step 6: Initialize the Cluster
This sets up the control plane. We use a specific CIDR range that Calico (our network provider) expects.

Bash

sudo kubeadm init --pod-network-cidr=192.168.0.0/16
Step 7: Configure Local Access
This allows your current user to run kubectl commands without needing sudo.

Bash

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Step 8: Install Calico (The Network)
Until you install a CNI (Container Network Interface), your pods won't be able to communicate.

Bash

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
🔌 Section C: Connect the Workers
Perform these steps ONLY on Worker nodes.

Step 9: Join the Cluster
On your Master node, run this to get your unique join command:

Bash

kubeadm token create --print-join-command
Copy the output and run it on your Worker nodes with sudo.

✅ Section D: Final Validation
Step 10: Verify the Cluster
Back on the Master node, check the status of your nodes:

Bash

kubectl get nodes
Expected Result: You should see your Master and Worker nodes listed as Ready.

📖 Common Fixes
Nodes stuck in NotReady: Give Calico about 2 minutes to initialize. Check status with kubectl get pods -n kube-system.

Kubelet won't start: Double-check that Swap is truly off (swapoff -a).