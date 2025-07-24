
# 📘 **Day 7: Liveness & Readiness Probes + Environment Variables**

---



## 🧠 1. Understanding **Probes**

### 🔹 **Liveness Probe**

> Checks **if a container is still running properly**
> If failed repeatedly → Kubernetes restarts the container

✅ Example: App gets stuck in infinite loop or deadlock

---

### 🔹 **Readiness Probe**

> Checks **if a container is ready to accept requests**
> If not ready → Pod won’t get traffic via Service

✅ Example: App is still booting or waiting for DB

---

### ✅ Types of Probes

| Type        | Description                 |
| ----------- | --------------------------- |
| `httpGet`   | Sends HTTP GET to container |
| `exec`      | Runs a shell command        |
| `tcpSocket` | Checks if TCP port open     |

---

### 📄 Example App with Probes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 3
      periodSeconds: 5
```

---

### 🧪 Commands

```bash
kubectl apply -f probe-demo.yaml
kubectl describe pod probe-demo
kubectl get pods
```

---

## 🧠 2. Understanding **Environment Variables**

### 🔹 Literal env variables

```yaml
env:
- name: APP_MODE
  value: "production"
```

---

### 🔹 From ConfigMap

```yaml
envFrom:
- configMapRef:
    name: my-config
```

---

### 🔹 From Secret

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
```

---

## ✅ Hands-on: Create a Pod with All Env Types

### Step 1: ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  APP_MODE: debug
  FEATURE_X: "enabled"
```

### Step 2: Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=    # base64 for "password"
```

---

### Step 3: Pod Using Them

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: demo-secret
          key: DB_PASSWORD
    envFrom:
    - configMapRef:
        name: demo-config
```

---

### 🧪 Run It

```bash
kubectl apply -f demo-config.yaml
kubectl apply -f demo-secret.yaml
kubectl apply -f env-demo.yaml

kubectl exec -it env-demo -- printenv | grep DB_PASSWORD
kubectl exec -it env-demo -- printenv | grep APP_MODE
```

---

## 🧹 Cleanup

```bash
kubectl delete pod probe-demo env-demo
kubectl delete configmap demo-config
kubectl delete secret demo-secret
```

---

## 📝 Recap

| Topic           | Key Insight                          |
| --------------- | ------------------------------------ |
| Liveness probe  | Restarts container if unhealthy      |
| Readiness probe | Keeps traffic off until app is ready |
| ConfigMap       | Inject env variables for config      |
| Secret          | Inject secure values into env        |

---


