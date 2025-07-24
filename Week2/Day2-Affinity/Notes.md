Absolutely — let’s elevate the quality to **expert-level**, with a more **structured**, **real-world** and **insightful** guide to Kubernetes **Affinity & Anti-Affinity**, as if you're preparing for **CKA** or managing **production clusters**.

---

# 🚀 Advanced Kubernetes Scheduling – Affinity & Anti-Affinity (Day 9)

---

## 🧭 Why Affinity Matters

Kubernetes default scheduling is powerful — but not always enough.

> Real-world production systems need pods **spread for high availability**, **grouped for performance**, or **assigned to specific hardware** like GPU nodes.

This is where Affinity, Anti-Affinity, and Node Affinity provide **intelligent control** over scheduling.

---

## 🔖 Overview of Affinity Types

| Type              | Targets         | Purpose                          | Topology Scope       |
| ----------------- | --------------- | -------------------------------- | -------------------- |
| `nodeAffinity`    | **Node labels** | Tie pods to specific nodes       | Node only            |
| `podAffinity`     | **Other pods**  | Co-locate with other pods        | Node / Zone / Region |
| `podAntiAffinity` | **Other pods**  | Avoid scheduling with other pods | Node / Zone / Region |

---

## 🧠 1. Node Affinity (Target Specific Nodes)

Node affinity allows **filtering nodes** based on their labels. It's like `nodeSelector`, but smarter — it supports **complex match rules**.

### 📘 Syntax

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

🧩 Key Concepts:

* `requiredDuringScheduling...`: **Mandatory**. If no node matches, pod stays `Pending`.
* `preferredDuringScheduling...`: **Best effort**. Scheduler tries, but falls back.

---

### 💡 Use Case: Run on GPU nodes

```yaml
nodeSelectorTerms:
  - matchExpressions:
      - key: kubernetes.io/instance-type
        operator: In
        values: [nvidia-gpu]
```

🔧 Label the node first:

```bash
kubectl label node <node-name> kubernetes.io/instance-type=nvidia-gpu
```

---

## 🧠 2. Pod Affinity (Schedule With Similar Pods)

**Co-location strategy**. Schedule pods *near* others with the same labels — used for:

* Latency-sensitive microservices
* Pods that need shared caching/locality

### 📘 Example

```yaml
affinity:
  podAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: frontend
        topologyKey: "kubernetes.io/hostname"
```

🧩 `topologyKey`: Scope of affinity:

* `"kubernetes.io/hostname"` → same node
* `"topology.kubernetes.io/zone"` → same zone

---

### 💡 Use Case: Keep microservice pair on same node

* `frontend` + `api-gateway`
* `cache` + `web`

Use same label like `group: web-tier`, then configure affinity.

---

## 🧠 3. Pod Anti-Affinity (Spread Across Nodes)

This is vital for **high availability**.

Use it to **ensure pods are NOT scheduled on the same node** (or even zone). Especially for:

* **StatefulSet pods**
* **Replicated databases** (MongoDB, Redis, RabbitMQ)
* **Web server clusters**

---

### 📘 Anti-Affinity Example

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: redis
        topologyKey: "kubernetes.io/hostname"
```

This ensures all `redis` pods land on **different nodes**.

---

## 🧪 Realistic Use Case: Highly Available Web App

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: web
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: web
          image: nginx
```

👀 Validate:

```bash
kubectl get pods -o wide
# Each pod should be on a different node
```

---

## ⚙️ How It Works Internally

* Kubernetes scheduler **filters nodes** based on affinity rules
* Then **prioritizes** based on weights (if `preferred`)
* **Fails scheduling** if required rules can't be met

---

## 🧱 Best Practices

| Tip                                   | Why It Matters                                 |
| ------------------------------------- | ---------------------------------------------- |
| Always label nodes meaningfully       | Enables clean nodeAffinity strategies          |
| Use Anti-Affinity for HA apps         | Avoids SPOF (single point of failure)          |
| Be specific with `topologyKey`        | Don’t accidentally co-locate in same zone/node |
| Prefer `preferredDuringScheduling...` | Prevent pods from getting stuck in `Pending`   |
| Combine affinity + taints/tolerations | Build robust multi-tenant node isolation       |

---

## ❌ Common Pitfalls

| Problem                          | Cause                                     | Fix                         |
| -------------------------------- | ----------------------------------------- | --------------------------- |
| Pods stuck in `Pending`          | No matching nodes for `required` rules    | Use `preferred` or relabel  |
| Affinity ignored                 | Label selector mismatch or missing labels | Double-check casing/labels  |
| Anti-affinity not spreading pods | Missing topologyKey or wrong label        | Set correct labels and keys |

---

## 🔍 Pro Tip: Mix Anti-Affinity with StatefulSet

```yaml
podAntiAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: kafka
      topologyKey: "kubernetes.io/hostname"
```

Ensures each Kafka pod in StatefulSet runs on its own node.

---

## 🧹 Cleanup for All Examples

```bash
kubectl delete deployment nginx-ha
kubectl delete deployment nginx-node-affinity
kubectl label node <node-name> disktype-
```

---

## 🎯 Summary – Affinity Rules Quick Guide

| Rule Type        | Target | Use Case                         | Key Field                   |
| ---------------- | ------ | -------------------------------- | --------------------------- |
| Node Affinity    | Node   | GPU, SSD, zone-specific hardware | nodeSelectorTerms           |
| Pod Affinity     | Pod    | Microservices together           | labelSelector + topologyKey |
| Pod AntiAffinity | Pod    | HA services, spread DBs/web pods | labelSelector + topologyKey |

---

