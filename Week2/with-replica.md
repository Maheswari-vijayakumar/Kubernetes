✅ **Replicas**
✅ **Taints & Tolerations**
✅ **Node Affinity**
✅ **Pod Anti-Affinity**

---

## 🧪 Scenario: Deploy 3 App Replicas with Smart Scheduling

### 🎯 Goal:

- Run 3 replicas of a pod
- Only on nodes **labeled** as `hardware=gpu`
- That are **tainted** with `gpu=true:NoSchedule`
- Spread the replicas **across different nodes** (anti-affinity)

---

## 🔧 Step 1: Prep the Cluster

### A. Label 2 or more nodes:

```bash
kubectl label nodes node1 hardware=gpu
kubectl label nodes node2 hardware=gpu
```

### B. Add taints to those nodes:

```bash
kubectl taint nodes node1 gpu=true:NoSchedule
kubectl taint nodes node2 gpu=true:NoSchedule
```

📌 You need at least **2 nodes** to see anti-affinity in action.

---

## 📄 Step 2: Deploy YAML (save as `gpu-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      tolerations:
        - key: "gpu"
          operator: "Equal"
          value: "true"
          effect: "NoSchedule"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: hardware
                    operator: In
                    values:
                      - gpu
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - gpu-app
              topologyKey: "kubernetes.io/hostname"
      containers:
        - name: app-container
          image: nginx
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
```

---

## 🚀 Step 3: Apply It

```bash
kubectl apply -f gpu-deployment.yaml
kubectl get pods -o wide
```

✅ You should see:

- Pods only running on `hardware=gpu` nodes
- No pods on other nodes
- No 2 pods on the **same node** (anti-affinity in action)

---

## 🧹 Step 4: Clean Up

```bash
kubectl delete deployment gpu-app
kubectl taint nodes node1 gpu=true:NoSchedule-
kubectl taint nodes node2 gpu=true:NoSchedule-
kubectl label nodes node1 hardware-
kubectl label nodes node2 hardware-
```

---

## 🧠 What You Practiced

| Concept           | In Action Here                         |
| ----------------- | -------------------------------------- |
| Taints            | Prevent scheduling unless allowed      |
| Tolerations       | Pod is allowed on tainted nodes        |
| Node Affinity     | Pod required node label `hardware=gpu` |
| Pod Anti-Affinity | Replicas spread across nodes           |

---

Would you like a **diagram** or **quiz** based on this deployment to reinforce what you learned?
