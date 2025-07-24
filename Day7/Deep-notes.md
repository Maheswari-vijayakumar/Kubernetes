# 📘 **Day 7 – Deep Dive: Liveness & Readiness Probes + Resource Requests & Limits**

---

## 🔹 PART 1: **Liveness & Readiness Probes**

---

### 🧠 What Are Probes?

| Probe Type        | Purpose                                        | Action Taken                                  |
| ----------------- | ---------------------------------------------- | --------------------------------------------- |
| `Liveness Probe`  | Checks if container is alive and running       | If fails → container is **restarted**         |
| `Readiness Probe` | Checks if container is ready to accept traffic | If fails → **removed from Service** endpoints |

---

### 🔎 Probe Types

| Type        | Description                         |
| ----------- | ----------------------------------- |
| `httpGet`   | Sends HTTP request to endpoint      |
| `tcpSocket` | Tries to open a TCP connection      |
| `exec`      | Executes a command in the container |

---

### 🔧 Key Fields

* `initialDelaySeconds`: Wait time before probe starts
* `periodSeconds`: Frequency of probe
* `failureThreshold`: Fail count before marking as failed
* `successThreshold`: Pass count before marking as passed

---

### 📄 **Example: Probes with NGINX**

📁 `probes-nginx.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-probes
spec:
  containers:
  - name: nginx
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

🧪 Test:

```bash
kubectl apply -f probes-nginx.yaml
kubectl describe pod nginx-probes
kubectl get endpoints nginx-probes  # Will not show until readiness passes
```

🧹 Cleanup:

```bash
kubectl delete -f probes-nginx.yaml
```

---

## 🔹 PART 2: **Resource Requests & Limits**

---

### 🧠 What Are Requests and Limits?

| Setting    | Description                                                                   |
| ---------- | ----------------------------------------------------------------------------- |
| `requests` | Minimum resources guaranteed for the container                                |
| `limits`   | Max resources allowed; usage above this is throttled (CPU) or killed (Memory) |

---

### 💡 Why Use Them?

* Prevent **resource starvation**
* Enable **bin packing** and autoscaling
* Improve cluster **QoS and reliability**

---

### ⚙️ Units

* **CPU**: `m` = millicores (e.g., `500m` = 0.5 vCPU)
* **Memory**: `Mi` or `Gi`

---

### 📄 **Example: Resource Limits with BusyBox**

📁 `resources-busybox.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-limits
spec:
  containers:
  - name: bb
    image: busybox
    command: ["sh", "-c", "while true; do echo hello; sleep 10; done"]
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

📦 Apply:

```bash
kubectl apply -f resources-busybox.yaml
kubectl describe pod busybox-limits
```

💻 If you have metrics-server:

```bash
kubectl top pod busybox-limits
```

🧹 Cleanup:

```bash
kubectl delete -f resources-busybox.yaml
```

---

## 🔎 Optional Bonus: Simulate Failures

### 📄 Faulty app with failing liveness

📁 `liveness-fail.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fail-app
spec:
  containers:
  - name: bb
    image: busybox
    command: ["sh", "-c", "sleep 10 && exit 1"]
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

Watch behavior:

```bash
kubectl apply -f liveness-fail.yaml
kubectl describe pod fail-app
```

💡 You’ll see restarts due to failed liveness check.

🧹

```bash
kubectl delete -f liveness-fail.yaml
```

---

## 🧠 Summary Table

| Concept         | Key Use Case                          |
| --------------- | ------------------------------------- |
| Liveness Probe  | Detect & restart hung apps            |
| Readiness Probe | Traffic control until app is ready    |
| Requests        | Guarantee minimum scheduling resource |
| Limits          | Prevent abuse, improve stability      |

---

