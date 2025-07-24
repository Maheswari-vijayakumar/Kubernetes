# 📘 **Day 5 – ConfigMaps & Secrets**


---

## 🧠 THEORY

### 📦 What is a **ConfigMap**?

* A **ConfigMap** is used to **inject configuration data** (like key-value pairs) into Kubernetes resources.
* It keeps configuration separate from the container image.
* Can be used to:

  * Set environment variables
  * Populate command-line arguments
  * Mount as files

---

### 🔐 What is a **Secret**?

* A **Secret** is used to **store sensitive data** like passwords, API keys, tokens.
* Encoded in **Base64** (not encrypted by default)
* Kubernetes restricts Secret visibility via RBAC and etcd encryption (optional but recommended)

---

## 🛠️ HANDS-ON PRACTICE

---

### ✅ 1. Create a ConfigMap

📄 **configmap.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  ENV: "production"
  DEBUG: "false"
  APP_PORT: "8080"
```

📦 Apply:

```bash
kubectl apply -f configmap.yaml
kubectl get configmap app-config -o yaml
```

---

### ✅ 2. Create a Secret

📌 First, encode your sensitive values using base64:

```bash
echo -n "admin" | base64         # YWRtaW4=
echo -n "s3cr3tpass" | base64    # czNjcjN0cGFzcw==
```

📄 **secret.yaml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: czNjcjN0cGFzcw==
```

📦 Apply:

```bash
kubectl apply -f secret.yaml
kubectl get secret db-secret -o yaml
```

---

## 🚀 3. Use ConfigMap and Secret in a Pod

📄 **pod-with-config-and-secret.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  containers:
    - name: nginx
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: db-secret
```

📦 Apply:

```bash
kubectl apply -f pod-with-config-and-secret.yaml
```

🔎 Verify environment variables injected:

```bash
kubectl exec -it secure-app -- env | grep -E 'ENV|DEBUG|username|password'
```

---

## ✅ 4. Mount ConfigMap/Secret as Files

### 📄 ConfigMap mounted as files:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-vol-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: app-config
```

### 📄 Secret mounted as files:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: secret-vol
          mountPath: /etc/secrets
          readOnly: true
  volumes:
    - name: secret-vol
      secret:
        secretName: db-secret
```

📦 Apply both:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f config-vol-pod.yaml
kubectl apply -f secret-vol-pod.yaml
```

---

## 🧹 Cleanup

```bash
kubectl delete -f configmap.yaml
kubectl delete -f secret.yaml
kubectl delete -f pod-with-config-and-secret.yaml
kubectl delete -f config-vol-pod.yaml
kubectl delete -f secret-vol-pod.yaml
```

---

## ✅ Summary Table

| Resource  | Purpose               | Storage Format |
| --------- | --------------------- | -------------- |
| ConfigMap | Non-sensitive configs | Plain text     |
| Secret    | Sensitive credentials | Base64 encoded |


