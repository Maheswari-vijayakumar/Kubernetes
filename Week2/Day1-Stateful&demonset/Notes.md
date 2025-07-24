# 📘 **Day 8: StatefulSets & DaemonSets in Kubernetes**

---

## 🚀 What You Will Learn

| Topic           | Description                                    |
| --------------- | ---------------------------------------------- |
| 🔹 StatefulSets | Run stateful apps like databases with identity |
| 🔹 DaemonSets   | Run a pod on **every node** (e.g., monitoring) |
| 🔧 Hands-on     | Deploy a database, and a log agent (Fluentd)   |
| 🧹 Cleanup      | Delete resources, remove PVCs, labels etc.     |

---

## 1️⃣ StatefulSets – What & Why?

### 🧠 What is a StatefulSet?

> A **StatefulSet** is a special controller used to manage **stateful applications**.

Unlike Deployments:

* Pods **get stable names** (like `mysql-0`, `mysql-1`)
* Each pod has its own **persistent volume** (PVC)
* **Orderly start/stop** of pods

### 🧰 Use Cases

* Databases: MySQL, PostgreSQL
* Queues: Kafka, RabbitMQ
* Clusters: Cassandra, Elasticsearch

---

### ✅ Hands-on: Deploy a simple MySQL StatefulSet

### 📄 `mysql-statefulset.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None   # Required for StatefulSet
  selector:
    app: mysql
  ports:
  - port: 3306
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

---

### 🧪 Apply it

```bash
kubectl apply -f mysql-statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=mysql -o wide
```

Each pod will be named:

```
mysql-0
mysql-1
```

---

### 📂 Check Persistent Volumes

```bash
kubectl get pvc
```

Each pod has its own PVC like:

```
mysql-data-mysql-0
mysql-data-mysql-1
```

---

## 🧹 Clean Up StatefulSet

```bash
kubectl delete -f mysql-statefulset.yaml
kubectl delete pvc -l app=mysql
```

---

## 2️⃣ DaemonSets – What & Why?

### 🧠 What is a DaemonSet?

> A **DaemonSet** ensures that one copy of a pod runs **on every node** (or a specific group of nodes).

### 🧰 Use Cases

* Log collectors: **Fluentd, Filebeat**
* Monitoring: **Prometheus Node Exporter**
* Network agents, firewall enforcers

---

### ✅ Hands-on: Deploy Fluentd on all nodes

### 📄 `fluentd-daemonset.yaml`

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
```

---

### 🧪 Apply It

```bash
kubectl apply -f fluentd-daemonset.yaml
kubectl get daemonsets
kubectl get pods -l app=fluentd -o wide
```

✅ You’ll see one pod running **on every node**.

---

### 🧹 Clean Up DaemonSet

```bash
kubectl delete -f fluentd-daemonset.yaml
```

---

## 📊 Comparison Table

| Feature             | StatefulSet        | DaemonSet               |
| ------------------- | ------------------ | ----------------------- |
| Pod identity        | Stable & ordered   | No, runs everywhere     |
| Storage             | Unique PVC per pod | Usually ephemeral       |
| Use case            | DBs, queues        | Logging, metrics agents |
| Naming              | `podname-0`, etc.  | Regular pod names       |
| Scheduling behavior | One per replica    | One per node            |

---

## ✅ Summary

* ✅ You now understand when to use **StatefulSet** vs **DaemonSet**
* ✅ You deployed a MySQL StatefulSet with PVCs
* ✅ You ran a Fluentd DaemonSet on every node


