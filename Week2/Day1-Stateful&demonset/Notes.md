# 🧠 Day 8 – Deep Dive: **StatefulSets & DaemonSets**

---

## 🔷 Part 1 – 🔍 **StatefulSets**

---

### 🧠 What Makes a Pod Stateful?

Unlike Deployments (ephemeral), **StatefulSets maintain identity across pod restarts**. This identity includes:

* **Consistent DNS**: `pod-0.myservice.default.svc.cluster.local`
* **Stable Storage**: Volume doesn't get destroyed on pod restart
* **Ordered Ops**: Create, delete, scale **one-by-one**

---

### 🧬 Components of a StatefulSet:

| Element                | Description                                                 |
| ---------------------- | ----------------------------------------------------------- |
| `volumeClaimTemplates` | Creates persistent volumes per pod (e.g., `web-0`, `web-1`) |
| `serviceName`          | Headless service — provides stable network ID               |
| `replicas`             | How many stateful pods to run                               |
| `podManagementPolicy`  | Default: `OrderedReady`, alt: `Parallel`                    |

---

### ✅ Use Cases for StatefulSet

* Databases: **MySQL**, **PostgreSQL**, **MongoDB**
* Distributed systems: **Kafka**, **Elasticsearch**, **Zookeeper**
* Any app that needs **stable storage** and **predictable names**

---

### 🧪 Hands-on: Deploy MongoDB StatefulSet

📁 `mongo-statefulset.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
spec:
  clusterIP: None  # Headless
  selector:
    app: mongo
  ports:
    - port: 27017
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
spec:
  serviceName: "mongo"
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:5.0
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongo-pvc
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongo-pvc
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 1Gi
```

📦 Deploy it:

```bash
kubectl apply -f mongo-statefulset.yaml
kubectl get pods -l app=mongo
kubectl get pvc
```

Check stable pod names:

```bash
kubectl get pods -o wide  # mongo-0, mongo-1, mongo-2
kubectl exec -it mongo-0 -- mongo  # Open Mongo shell
```

---

### 🛠️ Debug Tips

* If PVCs don’t bind: `kubectl describe pvc`
* If pods are stuck: `kubectl describe pod mongo-0`
* Watch the ordered startup:

  ```bash
  kubectl get pods -w
  ```

---

### 🧹 Cleanup

```bash
kubectl delete -f mongo-statefulset.yaml
```

---

## 🔶 Part 2 – 🔍 **DaemonSets**

---

### 🧠 Why DaemonSet?

A **DaemonSet ensures that one copy of a pod runs on each node** — ideal for system-level agents or collectors.

---

### ✅ Typical DaemonSet Use Cases:

| Use Case          | Example Tool                            |
| ----------------- | --------------------------------------- |
| Log collection    | Fluentd, Logstash                       |
| Monitoring agents | Prometheus Node Exporter, Datadog Agent |
| Network tools     | Calico, Cilium                          |
| Security agents   | Falco, Sysdig                           |

---

### 📁 Fluentd-style Logging DaemonSet (Simple)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-writer
spec:
  selector:
    matchLabels:
      app: logger
  template:
    metadata:
      labels:
        app: logger
    spec:
      containers:
      - name: logger
        image: busybox
        command: ["/bin/sh", "-c", "while true; do echo $(hostname) >> /var/log/nodes.log; sleep 10; done"]
        volumeMounts:
        - name: logs
          mountPath: /var/log
      volumes:
      - name: logs
        hostPath:
          path: /var/log
```

📦 Apply:

```bash
kubectl apply -f daemonset-logger.yaml
kubectl get pods -o wide
```

Check that there's **1 pod per node** (control-plane + workers).

---

### 🧠 Advanced: Run only on worker nodes

Use nodeSelector:

```yaml
spec:
  template:
    spec:
      nodeSelector:
        node-role.kubernetes.io/worker: ""
```

Or use `tolerations` for taints.

---

### 📦 Monitor logs:

```bash
# Access log file from host or via hostPath mount
```

---

### 🧹 Cleanup:

```bash
kubectl delete -f daemonset-logger.yaml
```

---


---

## ✅ Summary Table

| Object          | Best For                    | Key Feature                        |
| --------------- | --------------------------- | ---------------------------------- |
| **StatefulSet** | Databases, distributed apps | Stable identity, storage, DNS      |
| **DaemonSet**   | Log agents, monitors        | 1 pod per node, no replica control |

---

### ✅ You’re now ready for real-world workloads:

* MongoDB, RabbitMQ (StatefulSets)
* Fluentd, Prometheus Node Exporter (DaemonSets)

---

