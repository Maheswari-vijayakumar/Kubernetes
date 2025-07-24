# 🧪 **Day 14: Final Practice Lab — Full App Deployment with Ingress**

---

## 🎯 **Lab Goals**

You'll deploy a real-world-like app stack that includes:

| Feature               | Technology                  |
| --------------------- | --------------------------- |
| 🔁 App Deployment     | A simple NGINX web app      |
| 🔧 Config Injection   | Using **ConfigMap**         |
| 📦 Persistent Storage | With **PVC**                |
| 🌍 External Access    | Through **Ingress**         |
| 🧪 Health Probes      | Liveness & Readiness checks |
| 📉 Resource Limits    | CPU & Memory caps           |

---

## 📁 File Structure

```
day14-lab/
├── configmap.yaml
├── pvc.yaml
├── deployment.yaml
├── ingress.yaml
├── index.html
```

---

## 📄 1. Create ConfigMap (`configmap.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-config
data:
  index.html: |
    <html>
    <body>
      <h1>Hello from Kubernetes Lab!</h1>
    </body>
    </html>
```

---

## 📄 2. Create Persistent Volume Claim (`pvc.yaml`)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: html-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 📄 3. Create Deployment (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-vol
          mountPath: /usr/share/nginx/html
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
      volumes:
      - name: html-vol
        persistentVolumeClaim:
          claimName: html-pvc
```

> 💡 The volume mounts will be pre-populated using a Kubernetes **Init Container** (next step).

---

### 🧩 Optional: Populate PVC using Init Container (Optional Expert Tip)

If you want to preload the PVC with your ConfigMap HTML, we can add an **init container** — ask and I’ll guide you separately.

---

## 📄 4. Create Ingress (`ingress.yaml`)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: lab.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-web
            port:
              number: 80
```

---

## 🧩 5. Create Service for Deployment

Add this to a new file or inside `deployment.yaml`:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-web
spec:
  selector:
    app: nginx-web
  ports:
    - port: 80
      targetPort: 80
```

---

## 🚀 Deploy Everything

```bash
kubectl apply -f configmap.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml
```

---

## 🧪 Test the Ingress

Add to your `/etc/hosts`:

```
127.0.0.1 lab.local
```

Then open:

```
http://lab.local
```

You should see:

```
Hello from Kubernetes Lab!
```

---

## 🧹 Cleanup

```bash
kubectl delete ingress nginx-ingress
kubectl delete svc nginx-web
kubectl delete deploy nginx-web
kubectl delete pvc html-pvc
kubectl delete configmap web-config
```

---

## 🎉 What You Practiced Today

| Component        | Covered                           |
| ---------------- | --------------------------------- |
| 🧱 ConfigMap     | ✅ HTML template                   |
| 💽 PVC           | ✅ Persistent storage              |
| 🐳 Deployment    | ✅ With resource limits and probes |
| 🌐 Ingress       | ✅ External routing                |
| 🧪 Health Checks | ✅ Liveness + Readiness            |
| ⚙️ Service       | ✅ ClusterIP exposed to Ingress    |

---

