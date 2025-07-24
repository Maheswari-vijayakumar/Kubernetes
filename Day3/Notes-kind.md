# 📅 **Day 3 – Services & Networking (with KIND)**


---

## 🧰 Step 1: Create a KIND Cluster with Port Mapping

Create a config file to map the `NodePort` from the container to your host machine:

📄 **`kind-config.yaml`**

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 30007
      hostPort: 30007
- role: worker
- role: worker

```

🧪 **Run the command:**

```bash
kind create cluster --name k8s-networking --config kind-config.yaml
```

---

## 🛠️ Step 2: Deploy the Application (nginx)

📄 **`deployment.yaml`**

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
          image: nginx
```

📦 **Apply:**

```bash
kubectl apply -f deployment.yaml
```

---

## 🔁 Step 3: Create Services

---

### 1️⃣ **ClusterIP Service** (Internal only)

📄 **`clusterip.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

📦 Apply:

```bash
kubectl apply -f clusterip.yaml
```

🔎 Test from inside the cluster:

```bash
kubectl run test-curl --image=radial/busyboxplus:curl -it --rm --restart=Never -- curl nginx-clusterip
```

---

### 2️⃣ **NodePort Service**

📄 **`nodeport.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

📦 Apply:

```bash
kubectl apply -f nodeport.yaml
```

🌐 Access in browser or curl:

```bash
curl http://localhost:30007
```

---

### 3️⃣ **LoadBalancer Service** (Simulated)

📄 **`loadbalancer.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

📦 Apply:

```bash
kubectl apply -f loadbalancer.yaml
```

📡 Simulate LoadBalancer (KIND workaround):

```bash
kubectl port-forward service/nginx-loadbalancer 8080:80
```

Then test at:

```
http://localhost:8080
```

---

## 🧹 Step 4: Cleanup

```bash
kubectl delete -f clusterip.yaml
kubectl delete -f nodeport.yaml
kubectl delete -f loadbalancer.yaml
kubectl delete -f deployment.yaml
kind delete cluster --name k8s-networking
```

