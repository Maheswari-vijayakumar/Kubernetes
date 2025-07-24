# 🌐 **Ingress in Kubernetes — Beginner to Expert Guide**

---

## ✅ What You’ll Learn

| Level           | Topics                                                      |
| --------------- | ----------------------------------------------------------- |
| 🟢 Beginner     | What is Ingress, Ingress Controller, basic Ingress resource |
| 🟡 Intermediate | Path-based & host-based routing, services behind Ingress    |
| 🔴 Advanced     | TLS (HTTPS), annotations, rate limiting, custom error pages |
| ⚙️ Bonus        | NGINX Ingress installation, debugging tips, best practices  |

---

## 🟢 **1. What is Ingress? (Beginner)**

> **Ingress = Smart HTTP/HTTPS Router for your cluster**

### Without Ingress:

* You expose services using `NodePort` or `LoadBalancer`
* Expensive or clumsy in production

### With Ingress:

* One public IP for **many services**
* Control routing based on path, host
* Support TLS, authentication, and rate-limiting via **Ingress Controller**

---

## 💡 Ingress vs Ingress Controller

| Component              | Description                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------ |
| **Ingress Resource**   | Kubernetes object (`apiVersion: networking.k8s.io/v1`) — defines rules                     |
| **Ingress Controller** | Pod running in cluster that listens for Ingress and manages traffic (e.g., NGINX, Traefik) |

---

## 🧪 Hands-on: Install NGINX Ingress Controller (Kind Compatible)

### Step 1: Create a Cluster with Ingress Ports

```yaml
# kind-ingress-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 80
      hostPort: 80
    - containerPort: 443
      hostPort: 443
```

```bash
kind create cluster --config kind-ingress-cluster.yaml --name ingress-demo
```

---

### Step 2: Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/kind/deploy.yaml
```

Wait until pods are ready:

```bash
kubectl get pods -n ingress-nginx
```

---

## 🟡 **2. Create Basic Ingress Resource**

### App 1: Echo Server

```yaml
# echo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels:
        app: echo
    spec:
      containers:
      - name: echo
        image: ealen/echo-server
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: echo
spec:
  selector:
    app: echo
  ports:
  - port: 80
```

```bash
kubectl apply -f echo-deployment.yaml
```

---

### Create Ingress Resource

```yaml
# echo-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echo
            port:
              number: 80
```

```bash
kubectl apply -f echo-ingress.yaml
```

> Now access: [http://localhost/](http://localhost/) and you’ll see echo server response.

---

## 🔁 **3. Host-based & Path-based Routing**

### Example with Two Apps

* `/app1` → App1
* `/app2` → App2

```yaml
spec:
  rules:
  - host: "demo.local"
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

To access `http://demo.local`, add to `/etc/hosts`:

```
127.0.0.1 demo.local
```

---

## 🔐 **4. Add TLS (HTTPS) to Ingress**

### Step 1: Generate TLS Secret

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
 -keyout tls.key -out tls.crt -subj "/CN=demo.local"

kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key
```

---

### Step 2: Update Ingress

```yaml
spec:
  tls:
  - hosts:
    - demo.local
    secretName: tls-secret
```

> Now try: `https://demo.local`

---

## 🧠 Advanced Ingress Features

| Feature         | Description                   |
| --------------- | ----------------------------- |
| ✅ Auth          | Basic or JWT with annotations |
| ✅ Rate Limit    | Control request rate per IP   |
| ✅ CORS          | Cross-Origin support          |
| ✅ Custom Errors | Show friendly error pages     |
| ✅ Redirect      | Force HTTPS, strip path etc.  |

> All via [Ingress Annotations](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/)

---

## 🛠 Debugging Ingress

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl logs <nginx-pod-name> -n ingress-nginx
```

---

## 🧹 Cleanup

```bash
kubectl delete ingress echo-ingress
kubectl delete svc echo
kubectl delete deploy echo
kind delete cluster --name ingress-demo
```

---

## ✅ Summary

| Skill           | Mastered                        |
| --------------- | ------------------------------- |
| Basic Ingress   | ✅ Routing to services           |
| Host/Path-based | ✅ Multi-app routing             |
| TLS Setup       | ✅ Secure HTTPS with certs       |
| Advanced        | ✅ Auth, rate limit, error pages |
| NGINX Ingress   | ✅ Installed & configured        |


