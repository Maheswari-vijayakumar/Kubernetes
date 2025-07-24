# 📘 **Day 7 – Liveness & Readiness Probes + Resource Limits**

---

### 🧠 THEORY:

| Concept             | Purpose                                               |
| ------------------- | ----------------------------------------------------- |
| **Liveness Probe**  | Checks if app is still running — restart if it fails  |
| **Readiness Probe** | Checks if app is ready to serve traffic — hold off LB |
| **Resource Limits** | Define CPU & memory boundaries for containers         |

---

### 🛠️ HANDS-ON

#### ✅ 1. Liveness + Readiness Demo Pod

📄 **probes-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probes-demo
spec:
  containers:
  - name: app
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
      initialDelaySeconds: 2
      periodSeconds: 5
```

📦 Apply:

```bash
kubectl apply -f probes-pod.yaml
kubectl describe pod probes-demo
```

---

#### ✅ 2. Resource Limits

📄 **resources-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: limits-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do echo hello; sleep 10; done"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

📦 Apply:

```bash
kubectl apply -f resources-pod.yaml
kubectl top pod limits-demo  # if metrics-server is running
```

---

### 🧹 Cleanup

```bash
kubectl delete -f probes-pod.yaml
kubectl delete -f resources-pod.yaml
```

---

### ✅ Summary

| Feature             | Used for                             |
| ------------------- | ------------------------------------ |
| **Liveness Probe**  | Restarting stuck containers          |
| **Readiness Probe** | LoadBalancer/Service traffic control |
| **Resource Limits** | Scheduling & QoS classification      |

---

