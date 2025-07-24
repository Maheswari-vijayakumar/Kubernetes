Awesome! Let's dive into **Day 13: Rolling Updates & Rollbacks** — a must-know for managing safe, zero-downtime deployments in Kubernetes.

---

# 📘 **Day 13: Rolling Updates & Rollbacks**

---

## ✅ What You’ll Learn

| Topic                    | Description                                |
| ------------------------ | ------------------------------------------ |
| 🔁 Rolling Update        | Update Pods one at a time with no downtime |
| 🔙 Rollback              | Revert to previous version if things break |
| ⛓️ Deployment Strategies | Understand how Kubernetes handles updates  |
| 🔧 Hands-on              | Practice with `kubectl rollout`            |
| 🧹 Cleanup               | Remove all related resources               |

---

## 1️⃣ What is a Rolling Update?

A **Rolling Update**:

* Replaces old Pods with new ones **incrementally**
* Ensures some Pods are always available
* Default behavior for Kubernetes **Deployments**

---

### 💡 Example Scenario

You deploy version `1.0` of your app.
Later, you update the image to version `2.0`.
Kubernetes will:

* Spin up a new Pod with `v2.0`
* Wait until it’s healthy
* Then shut down an old Pod with `v1.0`
* Repeat until all Pods are updated

---

## 2️⃣ Rollback

If the new version fails (bugs, crashes, etc.), you can:

```bash
kubectl rollout undo deployment <deployment-name>
```

This reverts to the **last working configuration**.

---

## 🧪 Hands-on Lab

### Step 1: Create the Initial Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f myapp-deployment.yaml
kubectl get pods
```

---

### Step 2: Update the Image (Rolling Update)

```bash
kubectl set image deployment/myapp nginx=nginx:1.25 --record
```

* `--record` tells Kubernetes to save this change in rollout history

Check the rollout:

```bash
kubectl rollout status deployment/myapp
```

---

### Step 3: Rollback

Oops! Say version `1.25` is unstable. Roll back:

```bash
kubectl rollout undo deployment/myapp
```

Check the image:

```bash
kubectl describe deployment myapp | grep Image
```

---

### Step 4: Rollout History

```bash
kubectl rollout history deployment/myapp
```

You’ll see revisions, including recorded ones (if `--record` used).

---

## 3️⃣ Tune the Rollout Strategy

You can customize how fast or safe your rollout is:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1         # 1 extra pod above desired replicas
    maxUnavailable: 1   # 1 pod can be unavailable during update
```

---

## ⚠️ Common Errors

| Error                     | Fix                             |
| ------------------------- | ------------------------------- |
| `ImagePullBackOff`        | Wrong image tag or private repo |
| App crashes after rollout | Use `kubectl rollout undo`      |
| Pods not updated          | Check selector/label match      |

---

## 🧹 Cleanup

```bash
kubectl delete deployment myapp
```

---

## 📝 Summary

| Action            | Command                              |
| ----------------- | ------------------------------------ |
| Update image      | `kubectl set image`                  |
| Monitor rollout   | `kubectl rollout status`             |
| View history      | `kubectl rollout history`            |
| Undo change       | `kubectl rollout undo`               |
| Customize rollout | Use `strategy.rollingUpdate` in YAML |

---

## 💡 Challenge for You

1. Create a deployment with 4 replicas and version `1.17`
2. Update to version `1.21`
3. Watch rollout behavior with `kubectl rollout status`
4. Force a bad update (e.g., wrong image)
5. Rollback and fix it

---

