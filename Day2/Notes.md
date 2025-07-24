# 📅 **Day 2 – Pods, ReplicaSets & Deployments**

## 🎯 Goals for Today

* Understand the difference between **Pods**, **ReplicaSets**, and **Deployments**
* Create resources using both `kubectl` and **YAML**
* Practice scaling and updating applications

---

## 🧠 Theory

### 🔹 What is a **Pod**?

* The **smallest deployable unit** in Kubernetes
* Can run **one or more containers** (usually one)
* Shares **network namespace** and **storage** between containers in the same pod

### 🔹 What is a **ReplicaSet**?

* Ensures a specified number of **pod replicas** are running at all times
* Replaces crashed pods automatically
* Usually not used directly — **managed by Deployments**

### 🔹 What is a **Deployment**?

* Manages ReplicaSets and provides:

  * **Rolling updates**
  * **Rollbacks**
  * **Declarative configuration**
* Most common way to deploy apps

---

## 🛠️ Hands-on Lab

Create a folder structure:

```
kubernetes-learning/
└── day2/
        ├── pod.yaml
        ├── replicaset.yaml
        └── deployment.yaml
```

### ✅ Step 1: Create a Simple Pod

**pod.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod my-nginx
```

---

### ✅ Step 2: Create a ReplicaSet

**`replicaset.yaml`**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
```

```bash
kubectl apply -f replicaset.yaml
kubectl get rs
kubectl get pods -l app=nginx
```

---

### ✅ Step 3: Create a Deployment

**`deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.21
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl rollout status deployment nginx-deploy
```

---

## 🔄 Scaling & Updating

### Scale up the deployment:

```bash
kubectl scale deployment nginx-deploy --replicas=4
kubectl get pods
```

### Update image version:

```bash
kubectl set image deployment nginx-deploy nginx=nginx:1.23
kubectl rollout status deployment nginx-deploy
```

### Rollback:

```bash
kubectl rollout undo deployment nginx-deploy
```

---

## ✅ Day 2 Summary

| Command                           | Description                  |
| --------------------------------- | ---------------------------- |
| `kubectl apply -f`                | Apply YAML                   |
| `kubectl get rs/deployments/pods` | Inspect resources            |
| `kubectl scale deployment`        | Increase replicas            |
| `kubectl set image`               | Trigger rolling update       |
| `kubectl rollout undo`            | Rollback to previous version |

---

### Cleanup

```
kubectl delete -f deployment.yaml
kubectl delete -f replicaset.yaml
kubectl delete -f pod.yaml
```

```
kubectl delete pod my-nginx
kubectl delete rs nginx-rs
kubectl delete deployment nginx-deploy
```
