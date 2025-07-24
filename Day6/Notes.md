# 📘 **Day 5 – Volumes & Persistent Storage**


---

## 🧠 THEORY

### 🔹 Why Volumes?

* Containers are **ephemeral**: data inside them is lost after restart.
* Volumes provide **persistent storage** to pods.
* They can be:

  * Local (emptyDir, hostPath)
  * External (NFS, cloud disk, CSI)
  * Managed (PersistentVolume + PersistentVolumeClaim)

---

### 🔸 Volume Types:

| Type                          | Description                                      |
| ----------------------------- | ------------------------------------------------ |
| `emptyDir`                    | Temporary storage, deleted when pod dies         |
| `hostPath`                    | Mount a host machine's file path into the pod    |
| `PersistentVolume (PV)`       | Abstraction for real storage (local, cloud, NFS) |
| `PersistentVolumeClaim (PVC)` | A request to use a PV                            |

---

## 🛠️ PRACTICE

---

### ✅ 1. **EmptyDir Volume**

📄 **pod-emptydir.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo Hello from emptyDir > /data/hello.txt && sleep 3600"]
      volumeMounts:
        - name: temp-storage
          mountPath: /data
  volumes:
    - name: temp-storage
      emptyDir: {}
```

📦 Apply & Check:

```bash
kubectl apply -f pod-emptydir.yaml
kubectl exec -it pod-emptydir -- cat /data/hello.txt
```

---

### ✅ 2. **hostPath Volume (for local dev only)**

📄 **pod-hostpath.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: host-vol
          mountPath: /mnt/host
  volumes:
    - name: host-vol
      hostPath:
        path: /tmp/hostdata
        type: DirectoryOrCreate
```

📦 Apply & Check (on host system):

```bash
kubectl apply -f pod-hostpath.yaml
kubectl exec -it pod-hostpath -- sh -c "echo K8s > /mnt/host/test.txt"
cat /tmp/hostdata/test.txt  # run this on host machine (if on Linux)
```

⚠️ **Note:** On Windows with KIND, `hostPath` won’t work as expected. It’s better to stick to `emptyDir` or use `localPath-provisioner` or external PVCs.

---

### ✅ 3. **PersistentVolume (PV) & PersistentVolumeClaim (PVC)**

#### 🧱 Create a PV (local path example)

📄 **pv.yaml**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/data  # Works on Linux host or WSL
  persistentVolumeReclaimPolicy: Retain
```

#### 📦 Create a PVC

📄 **pvc.yaml**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply both:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl get pv,pvc
```

---

### ✅ 4. Use PVC in a Pod

📄 **pod-pvc.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-pvc
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - name: storage
          mountPath: /app/data
  volumes:
    - name: storage
      persistentVolumeClaim:
        claimName: local-pvc
```

📦 Apply & Check:

```bash
kubectl apply -f pod-pvc.yaml
kubectl exec -it pod-with-pvc -- sh -c "echo from PVC > /app/data/file.txt"
```

Check on host (if Linux):

```bash
cat /tmp/data/file.txt
```

---

## 🧹 Cleanup

```bash
kubectl delete -f pod-emptydir.yaml
kubectl delete -f pod-hostpath.yaml
kubectl delete -f pod-pvc.yaml
kubectl delete -f pvc.yaml
kubectl delete -f pv.yaml
```

---

## ✅ Summary Table

| Type       | Persistent | Notes                                |
| ---------- | ---------- | ------------------------------------ |
| `emptyDir` | ❌          | Removed when pod is deleted          |
| `hostPath` | ✅          | Tied to node's local path (dev only) |
| `PV + PVC` | ✅✅         | Dynamic or static provisioning       |

---


