
---

# 📅 **Day 1 – Introduction to Kubernetes & Setup**

## 🎯 Goals for Today

* Understand what Kubernetes is and why it's important
* Learn the core components of Kubernetes architecture
* Set up your local Kubernetes environment (Minikube or Kind)
* Run your first pod using `kubectl`

---

## 📘 Theory

### ✅ What is Kubernetes?

Kubernetes is an open-source **container orchestration** system for automating:

* Deployment
* Scaling
* Management
* Networking of containerized applications

It was originally developed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF).

---

### ✅ Why Use Kubernetes?

* **Automated scaling and self-healing** of applications
* **Rolling updates & rollbacks**
* **Service discovery and load balancing**
* **Multi-cloud and hybrid deployments**
* **Infrastructure as Code** via YAML

---

### ✅ Kubernetes Core Architecture

| Component              | Role                                                       |
| ---------------------- | ---------------------------------------------------------- |
| **Master Node**        | Control plane: orchestrates the cluster                    |
| **API Server**         | Front door to Kubernetes, handles RESTful operations       |
| **Scheduler**          | Assigns workloads to nodes                                 |
| **Controller Manager** | Reconciliation loops for Deployments, Replicas, etc.       |
| **etcd**               | Key-value store (cluster state)                            |
| **Worker Node**        | Runs the application Pods                                  |
| **Kubelet**            | Agent that runs on each node, communicates with API server |
| **Container Runtime**  | e.g., containerd, Docker — runs containers                 |
| **Kube-proxy**         | Manages networking for Pods (ClusterIP, Services)          |

---

## 🛠️ Hands-on Lab

### ✅ Step 1: Install Tools

You need:

* **kubectl** (CLI to control Kubernetes)
* **Minikube** (for single-node cluster) *or* **Kind** (Kubernetes IN Docker)


```

---

## 📜 `setup-k8s.ps1` — PowerShell Automation Script

This script installs `kubectl`, `Minikube`, and `Kind`, then starts a basic cluster.  
> ⚠️ **Run PowerShell as Administrator**

```powershell
# setup-k8s.ps1

# Install Chocolatey if needed
if (-not (Get-Command choco -ErrorAction SilentlyContinue)) {
    Set-ExecutionPolicy Bypass -Scope Process -Force
    [System.Net.ServicePointManager]::SecurityProtocol = 'Tls12'
    iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
}

# Install Docker Desktop (user must open it manually after install)
choco install docker-desktop -y

# Install kubectl
choco install kubernetes-cli -y


# => go with kind or minikube, let me go with kind
# Install Minikube 
# choco install minikube -y

# Install Kind
choco install kind -y

# Start Minikube (default Docker driver)
# Write-Host "Starting Minikube..."
# minikube start --driver=docker

# Confirm cluster status
kubectl version --client
kubectl cluster-info
kubectl get nodes

# Run a test pod
kubectl run nginx --image=nginx
kubectl get pods

````



### 💡 You can run it via PowerShell:

```
Set-ExecutionPolicy Bypass -Scope Process -Force
.\setup-k8s.ps1

```

### Inspect the pod:

```bash
kubectl describe pod nginx
kubectl logs nginx
kubectl exec -it nginx -- /bin/bash
```

### Expose the pod:

* **For Minikube**

  ```bash
  kubectl expose pod nginx --port=80 --type=NodePort
  minikube service nginx
  ```

* **For Kind**
  Kind doesn’t support `minikube service`, so:

  ```bash
  kubectl expose pod nginx --port=80 --type=NodePort
  kubectl get svc nginx
  ```

  Then forward the port:

  ```bash
  kubectl port-forward svc/nginx 8080:80
  # Now access via http://localhost:8080
  ```


