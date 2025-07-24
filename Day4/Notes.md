
# 📘 Day 4 – **Namespaces & Labels/Selectors**


---

## 🧠 THEORY

### 🔷 What is a **Namespace** in Kubernetes?

* Namespaces are virtual clusters backed by the same physical cluster.
* They allow segmentation of resources for **multi-tenant** or **multi-environment** setups.
* Useful in dividing workloads such as `dev`, `test`, `prod`, or even by team/project.

#### ✅ Common Namespaces:

| Name              | Description                                   |
| ----------------- | --------------------------------------------- |
| `default`         | Used when no namespace is specified           |
| `kube-system`     | System components (kube-dns, controller, etc) |
| `kube-public`     | Publicly readable, used for bootstrapping     |
| `kube-node-lease` | Node heartbeats for availability tracking     |

---

### 🔶 What are **Labels**?

* Key-value pairs attached to Kubernetes objects (Pods, Services, etc.).
* Help in **identifying**, **grouping**, and **selecting** objects logically.
* Labels can represent app versions, environments, teams, regions, etc.

#### Example:

```yaml
labels:
  app: frontend
  env: staging
  version: v1
```

---

### 🔶 What are **Label Selectors**?

* Mechanism to **filter or group resources** by matching labels.
* Used by Services, Deployments, NetworkPolicies, etc.

#### Types of Selectors:

* **Equality-based:** `app=frontend`
* **Set-based:** `app in (frontend, backend)`

---

## 🛠 HANDS-ON PRACTICE

---

### ✅ 1. Create a Namespace

📄 **namespace.yaml**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

📦 Apply:

```bash
kubectl apply -f namespace.yaml
kubectl get namespaces
```

---

### ✅ 2. Create a Pod with Labels

📄 **labeled-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  namespace: dev
  labels:
    app: frontend
    tier: web
    version: v1
spec:
  containers:
  - name: nginx
    image: nginx
```

📦 Apply:

```bash
kubectl apply -f labeled-pod.yaml
kubectl get pods -n dev --show-labels
```

---

### ✅ 3. Filter Using Label Selectors

```bash
# Get pods with label app=frontend
kubectl get pods -n dev -l app=frontend

# Get pods with multiple matching labels
kubectl get pods -n dev -l 'app=frontend,tier=web'
```

---

### ✅ 4. Use Labels in a Service (Selector)

📄 **service-selector.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: dev
spec:
  selector:
    app: frontend
    tier: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

📦 Apply:

```bash
kubectl apply -f service-selector.yaml
kubectl get svc -n dev
```

---

### 🧹 Cleanup

```bash
kubectl delete -f service-selector.yaml
kubectl delete -f labeled-pod.yaml
kubectl delete -f namespace.yaml
```

---

## ✅ Summary

| Concept       | Purpose                            |
| ------------- | ---------------------------------- |
| **Namespace** | Isolate environments and resources |
| **Label**     | Tag and group Kubernetes objects   |
| **Selector**  | Target objects by label filters    |


