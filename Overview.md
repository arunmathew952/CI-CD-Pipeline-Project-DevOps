\# 🚀 End-to-End DevSecOps CI/CD Pipeline on AWS + Kubernetes



This project demonstrates a \*\*production-style DevSecOps architecture\*\* that automates the complete software delivery lifecycle — from code commit to deployment and monitoring.



It integrates \*\*Cloud Infrastructure, CI/CD, Security, Containers, Kubernetes, and Observability\*\* into one working system.



---



\## 🧱 Architecture Overview



\*\*Flow:\*\*



Code → CI → Security Scans → Artifact Repository → Docker Image → Registry → Kubernetes Deployment → Monitoring



\*\*Tech Stack\*\*



| Category | Tools Used |

|---------|------------|

| Cloud | AWS EC2 |

| OS | Ubuntu Server |

| CI/CD | Jenkins |

| Build Tool | Maven |

| Code Quality | SonarQube |

| Security Scanning | Trivy |

| Artifact Repo | Nexus |

| Containerization | Docker |

| Orchestration | Kubernetes (kubeadm) |

| Networking | Calico CNI |

| Monitoring | Prometheus, Node Exporter, Blackbox Exporter |

| Visualization | Grafana |



---



\## ☁️ Phase 1 — AWS Infrastructure



Ubuntu EC2 instances were provisioned to serve as Kubernetes nodes.



\*\*Purpose:\*\*

\- Scalable compute

\- Secure SSH access

\- Base for Kubernetes cluster



---



\## ⚙️ Phase 2 — Kubernetes Cluster Setup



Cluster built using \*\*kubeadm\*\*.



Configured:



\- `containerd` (container runtime)

\- `kubeadm`, `kubelet`, `kubectl`

\- \*\*Calico CNI\*\* for networking



Result: A \*\*multi-node Kubernetes cluster\*\* ready for application deployment.



---



\## 💻 Phase 3 — Source Code Management



\- Code stored in a \*\*private Git repository\*\*

\- Authentication handled using \*\*Personal Access Tokens\*\*



---



\## 🔁 Phase 4 — CI/CD Pipeline (Jenkins)



Jenkins automates the full DevSecOps workflow.



\### 🔄 Pipeline Stages



1\. Git Checkout  

2\. Compile (Maven)  

3\. Unit Testing  

4\. File System Security Scan (Trivy)  

5\. SonarQube Code Analysis  

6\. Quality Gate Validation  

7\. Build Application Artifact  

8\. Publish Artifact to Nexus  

9\. Build Docker Image  

10\. Scan Docker Image (Trivy)  

11\. Push Image to Docker Registry  

12\. Deploy to Kubernetes  

13\. Verify Pods \& Services  

14\. Email Notifications with Reports  



✔ Ensures only \*\*tested, secure, and quality-approved code\*\* reaches production.



---



\## 🔒 DevSecOps Integration



Security is embedded into the pipeline:



| Tool | Purpose |

|------|---------|

| Trivy | Vulnerability scanning (FS + Docker images) |

| SonarQube | Code quality \& security analysis |

| Quality Gates | Automatic build blocking |



---



\## 📦 Containerization \& Deployment



Application packaged as a Docker image and deployed using:



```bash

kubectl apply -f deployment-service.yaml



