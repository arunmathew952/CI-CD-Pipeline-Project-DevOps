# Phase-1 | Step-1: Creating an Ubuntu EC2 Instance on AWS

## Objective
The objective of this step is to provision an Ubuntu-based EC2 instance on AWS, which will act as the base infrastructure for setting up a Kubernetes cluster using `kubeadm`.

This step focuses only on **infrastructure provisioning**, not Kubernetes installation.

---

## Prerequisites
- An active AWS account
- Basic familiarity with AWS Management Console
- SSH client (Mobaxterm / terminal)
- Key pair for EC2 access

---

## Step-by-Step Procedure

### 1. Sign in to AWS Management Console
- Navigate to: https://aws.amazon.com/console/
- Log in using AWS account credentials

---

### 2. Navigate to EC2 Dashboard
- In the AWS Console search bar, type **EC2**
- Select **EC2** under the *Compute* section
- Click **Instances** from the left sidebar

---

### 3. Launch a New EC2 Instance
- Click **Launch Instance**

---

### 4. Choose an Amazon Machine Image (AMI)
- Select **Ubuntu**
- Choose **Ubuntu Server 20.04 LTS** (or compatible LTS version)
- This OS is preferred due to its stability and Kubernetes compatibility

---

### 5. Choose Instance Type
- Select an instance type based on requirements  
  Recommended for practice:
  - `t2.micro` (testing)
  - `t3.medium` or higher (Kubernetes master/worker nodes)
- Click **Next: Configure Instance Details**

---

### 6. Configure Instance Details
- Leave default settings unless specific networking or IAM roles are required
- Ensure the instance is launched in the correct VPC and subnet
- Click **Next: Add Storage**

---

### 7. Add Storage
- Default root volume size is sufficient for basic Kubernetes setup
- Increase storage if required for logs or additional components
- Click **Next: Add Tags**

---

### 8. Add Tags (Optional but Recommended)
Example:
- Key: `Name`
- Value: `k8s-master` or `k8s-worker`

Tags help in identifying resources easily.

---

### 9. Configure Security Group
- Allow **SSH (port 22)** from your IP address
- Additional ports may be opened later if required (e.g., 6443 for Kubernetes API)

Example rules:
- SSH — TCP — Port 22 — Source: My IP
![Security Group SSH Rule](images/security-group-ssh.png)
---

### 10. Review and Launch
- Review all instance configurations
- Click **Launch**

---

### 11. Select or Create Key Pair
- Choose an existing key pair or create a new one
- Download and store the `.pem` file securely
- Confirm acknowledgment
- Click **Launch Instances**

---

### 12. Access the Instance
- Use **Mobaxterm** or SSH client to connect
- Login using:
  - Username: `ubuntu`
  - Authentication: EC2 key pair

---

## Outcome
At the end of this step:
- An Ubuntu EC2 instance is successfully created
- The instance is accessible via SSH
- The system is ready for Kubernetes prerequisite installation

---

## Next Step
➡️ **Phase-1 | Step-2: Installing Docker and Kubernetes Components**
