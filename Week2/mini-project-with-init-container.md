# ⚙️ Use Case: Inject HTML into PVC via Init Container

Kubernetes **does not** support mounting a ConfigMap directly into a PVC.
But… an **Init Container** can solve that:

1. Mount both the ConfigMap and the PVC as volumes.
2. Copy contents from the ConfigMap to the PVC.
3. The main container then uses the PVC as its data source.

---

## 📁 Directory Structure

```
day14-init/
├── configmap.yaml
├── pvc.yaml
├── deployment.yaml
├── ingress.yaml
```

---

## 📄 1. ConfigMap (HTML content) – `configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: html-config
data:
  index.html: |
    <html>
      <body>
        <h2>🔥 Init Container Injection Successful!</h2>
      </body>
    </html>
```

---

## 📄 2. PVC – `pvc.yaml`

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

## 📄 3. Deployment with Init Container – `deployment.yaml`

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
      initContainers:
      - name: init-copy-html
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
          - cp /config/index.html /usr/share/nginx/html/index.html
        volumeMounts:
        - name: config-vol
          mountPath: /config
        - name: html-vol
          mountPath: /usr/share/nginx/html

      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html-vol
          mountPath: /usr/share/nginx/html
      volumes:
      - name: config-vol
        configMap:
          name: html-config
      - name: html-vol
        persistentVolumeClaim:
          claimName: html-pvc
```

> 💡 `busybox` is used here to copy `index.html` from the config volume to the PVC mount.

---

## 📄 4. Ingress (Optional) – `ingress.yaml`

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
  - host: init.local
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

## 📄 5. Service – Add alongside deployment

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

## 🚀 Deploy All

```bash
kubectl apply -f configmap.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f ingress.yaml
```

Edit `/etc/hosts`:

```
127.0.0.1 init.local
```

Then open:

```
http://init.local
```

✅ You should see: “🔥 Init Container Injection Successful!”

---

## 🧹 Cleanup

```bash
kubectl delete -f .
```

---

