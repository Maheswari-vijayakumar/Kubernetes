# 🚀 Blue-Green vs Canary Deployments in Kubernetes

---

## 🔹 1. **Blue-Green Deployment**

### 🔧 What Is It?

You maintain two versions of the application:

* **Blue (v1)**: Currently serving users
* **Green (v2)**: New version deployed alongside

Once green is ready, **traffic is switched** (via a Service or Ingress) from **Blue ➝ Green** instantly.

---

### ✅ Benefits

* Instant rollback (just switch traffic back)
* Simple strategy
* Less risky for downtime

---

### ❌ Drawbacks

* Doubles resource usage temporarily
* Manual traffic switch unless automated

---

### 🔧 Example

#### Step 1: Deploy blue version

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

#### Step 2: Deploy green version

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

#### Step 3: Service points to blue initially

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue
  ports:
  - port: 80
    targetPort: 80
```

#### ✅ Switch traffic to green:

Update the service `selector` to:

```yaml
selector:
  app: myapp
  version: green
```

🎯 **Done!** — You just performed a Blue-Green switch.

---

## 🔸 2. **Canary Deployment**

### 🔧 What Is It?

You release the new version to a **small subset of users**, monitor for issues, then **gradually increase** traffic.

---

### ✅ Benefits

* Safer than all-at-once updates
* Real-world testing with small risk
* Can automate with metrics and rollback

---

### ❌ Drawbacks

* Requires smart routing (Ingress or Service mesh)
* Slightly more complex setup

---

### 🔧 Basic Example Using Replica Weights

Let’s assume we have 10 replicas:

* 9 use the **stable** version
* 1 uses the **canary** version

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

#### Shared Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector:
    app: myapp
  ports:
  - port: 80
```

This sends traffic **randomly** across all pods (weighted 9:1).

---

## 🧠 Advanced Canary: Use Ingress or Service Mesh

With tools like:

* **NGINX Ingress** (use annotations for traffic weights)
* **Istio / Linkerd** – precise control over traffic percentage, metrics, and automated rollback

---

## 🧹 Cleanup

```bash
kubectl delete deployment myapp-blue myapp-green myapp-stable myapp-canary
kubectl delete service myapp-service myapp
```

---

## ✅ Summary

| Strategy       | Risk   | Control     | Rollback | Use case                        |
| -------------- | ------ | ----------- | -------- | ------------------------------- |
| Rolling Update | Medium | Auto        | Easy     | Default safe strategy           |
| Blue-Green     | Low    | Manual/Auto | Instant  | Critical apps, prod env         |
| Canary         | Low    | High        | Gradual  | Large user base, phased rollout |

