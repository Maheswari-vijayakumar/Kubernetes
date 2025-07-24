### 🧩 Scenario:

- You have two apps:

  - `backend`: core business logic
  - `frontend`: UI that frequently calls backend

- You want **frontend pods to be placed near backend pods** (e.g., same node or zone)
- This improves **latency** and **data locality**

---

## 🛠️ Step-by-Step Breakdown

---

### 🔹 Step 1: Label the Nodes (optional but helpful for larger clusters)

> (We’ll use the default node topology for now — hostname)

```bash
kubectl label nodes <your-node-name> kubernetes.io/hostname=<node-hostname>
```

You don’t need this manually if you're using real nodes — the `kubernetes.io/hostname` label already exists.

---

### 🔹 Step 2: Deploy the `backend`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: nginx # placeholder
```

✅ This deploys 2 backend pods, labeled `app=backend`.

---

### 🔹 Step 3: Deploy the `frontend` with **Pod Affinity**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: backend
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: frontend
          image: nginx # placeholder
```

---

### 🔍 Explanation

- `podAffinity`: says “I want to be scheduled near pods with `app=backend`”
- `topologyKey: kubernetes.io/hostname`: restricts to the **same node**

So, Kubernetes tries to place each `frontend` pod on the **same node as a backend pod**.

---

### 🧪 How to Test

1. Apply both manifests:

   ```bash
   kubectl apply -f backend.yaml
   kubectl apply -f frontend.yaml
   ```

2. Check pod placement:

   ```bash
   kubectl get pods -o wide
   ```

✅ You should see **frontend pods scheduled near backend pods** (same node names).

---

### 🧠 Bonus: Use `preferredDuringScheduling...` for **soft affinity**

If you don’t want to force the placement (just prefer it), use:

```yaml
affinity:
  podAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: backend
          topologyKey: "kubernetes.io/hostname"
```

This makes it **optional** but preferred.

---

Would you like a downloadable `frontend.yaml` + `backend.yaml` zip or combined file? Or want to extend this with **anti-affinity for spreading replicas**?
