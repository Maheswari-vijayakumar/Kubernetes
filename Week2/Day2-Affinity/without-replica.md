### 🛠️ 1. Label and Taint the Node

```bash
kubectl label node <your-node-name> hardware=gpu
kubectl taint node <your-node-name> gpu=true:NoSchedule
```

📌 Replace `<your-node-name>` with the name of the node (`kubectl get nodes`).

---

### 📄 2. Create Pod YAML

Save as `gpu-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-task
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
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
```

---

### 🚀 3. Apply the Pod

```bash
kubectl apply -f gpu-pod.yaml
kubectl get pod gpu-task -o wide
```

✅ You’ll see it was scheduled **only on the GPU node** — because it both:

- **Tolerates** the taint ✅
- **Prefers** the hardware label via affinity ✅

---

### 🧹 4. Clean Up

```bash
kubectl delete pod gpu-task
kubectl taint node <your-node-name> gpu=true:NoSchedule-
kubectl label node <your-node-name> hardware-
```

---

## 🖼️ VISUAL DIAGRAM (Text-based)

```text
+-----------------------------+
|         Node A             |  <-- Tainted: gpu=true:NoSchedule
| Label: hardware=gpu        |
+-----------------------------+
            ▲
            | Affinity (must match hardware=gpu)
            | Toleration (must match taint gpu=true)
            ▼
    +----------------------+
    |     Pod: gpu-task    |
    +----------------------+
```

---
