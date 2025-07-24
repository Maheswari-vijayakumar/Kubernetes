# 🚀 30-Day Kubernetes Learning Plan

> 📌 **Daily Time Commitment**: \~2–3 hours
> 💡 Assumes basic familiarity with Linux/Docker (I can adjust if not)

---

## 🔰 Week 1: Introduction, Core Concepts, and Basics

| Day       | Topics                                                                     | Hands-on                                                                              |
| --------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Day 1** | What is Kubernetes, architecture overview (Nodes, Pods, Master components) | Install **Minikube** or **Kind**, install `kubectl`, explore Kubernetes dashboard     |
| **Day 2** | Pods, Containers in K8s, kubectl basics                                    | Run your first Pod with `kubectl run`, inspect with `get`, `describe`, `logs`, `exec` |
| **Day 3** | ReplicaSet and Deployment                                                  | Create a Deployment, scale up/down, update image, rollback                            |
| **Day 4** | Services (ClusterIP, NodePort, LoadBalancer)                               | Expose app using each service type, test endpoints                                    |
| **Day 5** | Namespaces & Labels/Selectors                                              | Create and switch namespaces, filter resources with labels                            |
| **Day 6** | ConfigMap and Secrets                                                      | Externalize config, mount as env vars and files                                       |
| **Day 7** | Volumes, PVCs, StorageClass (basics)                                       | Create and mount PVCs in Pods, use ephemeral and persistent storage                   |

---

## 🔧 Week 2: Core Workloads & Networking

| Day        | Topics                                             | Hands-on                                                       |
| ---------- | -------------------------------------------------- | -------------------------------------------------------------- |
| **Day 8**  | StatefulSets & DaemonSets                          | Deploy a DB with StatefulSet, run logging agent with DaemonSet |
| **Day 9**  | Jobs & CronJobs                                    | Schedule batch tasks and time-based jobs                       |
| **Day 10** | Ingress Basics & NGINX Ingress Controller          | Install Ingress Controller, set up routes                      |
| **Day 11** | Environment Variables, Probes (Liveness/Readiness) | Add probes to an app, restart on failure                       |
| **Day 12** | Resource Requests & Limits                         | Apply CPU/memory limits, test scaling behavior                 |
| **Day 13** | Rolling Update & Rollback                          | Use `kubectl rollout`, simulate failures and rollback          |
| **Day 14** | Review & Lab Day                                   | Do a practice project: NGINX + ConfigMap + PVC + Ingress       |

---

## 🚀 Week 3: Helm, RBAC, Monitoring, and Autoscaling

| Day        | Topics                                      | Hands-on                                                   |
| ---------- | ------------------------------------------- | ---------------------------------------------------------- |
| **Day 15** | Intro to Helm                               | Install Helm, deploy a Helm chart (e.g., nginx, WordPress) |
| **Day 16** | Writing your own Helm chart                 | Package a simple app using templates                       |
| **Day 17** | RBAC (Roles, RoleBindings, ServiceAccounts) | Limit access per namespace, test user permissions          |
| **Day 18** | Network Policies                            | Restrict Pod communication using Calico or Cilium          |
| **Day 19** | Metrics Server & HPA                        | Install metrics server, scale app using CPU usage          |
| **Day 20** | Prometheus & Grafana (via Helm)             | Set up monitoring and create a custom dashboard            |
| **Day 21** | Logging: EFK Stack                          | Deploy Elasticsearch + Fluentd + Kibana with Helm          |

---

## 🧠 Week 4: Advanced Topics, GitOps, Security & Certification Prep

| Day        | Topics                                                 | Hands-on                                                              |
| ---------- | ------------------------------------------------------ | --------------------------------------------------------------------- |
| **Day 22** | Operators & CRDs                                       | Install an operator (e.g., Redis), view CRDs                          |
| **Day 23** | CI/CD + GitHub Actions / Jenkins                       | Deploy app to K8s from Git push                                       |
| **Day 24** | GitOps with ArgoCD (or FluxCD)                         | Install ArgoCD, auto-sync from Git                                    |
| **Day 25** | PodSecurityContext, SecurityContext, PodSecurityPolicy | Restrict privileges, use non-root user                                |
| **Day 26** | Secrets Management (Vault / KMS intro)                 | Integrate K8s with HashiCorp Vault (basic)                            |
| **Day 27** | Multi-cluster setup (theory), Federation overview      | Discuss federation tools, cluster mesh concepts                       |
| **Day 28** | Service Mesh (Istio basics)                            | Install Istio, enable mTLS and traffic shifting                       |
| **Day 29** | CKA/CKAD Mock Test + Troubleshooting                   | Use killer.sh or practice labs (or I can provide a custom lab)        |
| **Day 30** | Final Project: Full App Deployment                     | Deploy a multi-service app with Helm, Ingress, monitoring, and GitOps |

---

## 🧰 Tools You’ll Use

* **Kubernetes tools**: `kubectl`, `k9s`, `kubectx`, `kubens`
* **Visualization**: Lens IDE, K8s Dashboard
* **Monitoring**: Prometheus, Grafana
* **Logging**: EFK/ELK
* **Helm**: Chart packaging
* **CI/CD**: GitHub Actions, Jenkins
* **GitOps**: ArgoCD, FluxCD

---

## 📘 Optional Add-ons

If you're aiming for **CKA/CKAD**, let me know—I can include:

* Practice YAML questions
* Bash shortcuts and cheatsheets
* Common troubleshooting labs

---


