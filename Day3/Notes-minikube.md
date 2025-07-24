# 📅 Day 3 – Services & Networking

## 🎯 Goals

* Understand how Kubernetes **exposes applications**
* Use **ClusterIP**, **NodePort**, and **LoadBalancer**
* Explore **DNS-based service discovery**
* Learn about **Service Selectors** and **Endpoints**

---

## 🧠 Concepts

### 🔌 What is a Service?

A **Service** is a stable endpoint for accessing a set of Pods. Pods are ephemeral — services provide a consistent way to access them.

**Types of Services:**

| Type           | Description                                          |
| -------------- | ---------------------------------------------------- |
| `ClusterIP`    | Internal cluster-only access (default)               |
| `NodePort`     | Exposes the service on a static port on each Node    |
| `LoadBalancer` | Exposes service externally using cloud provider’s LB |

---

## 🧪 Hands-on: Exposing a Deployment

### Step 1: Create a Deployment

Re-use Day 2's `deployment.yaml` or define one:

```bash
kubectl apply -f deployment.yaml
```

### Step 2: Create a ClusterIP Service

**`clusterip.yaml`**

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

```bash
kubectl apply -f clusterip.yaml
kubectl get svc
```

Try inside the cluster:

```bash
kubectl run curl --image=radial/busyboxplus:curl -it --rm --restart=Never -- curl nginx-clusterip
```

---

### Step 3: Create a NodePort Service

**`nodeport.yaml`**

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

```bash
kubectl apply -f nodeport.yaml
minikube service nginx-nodeport --url
```

---

### Step 4: Create a LoadBalancer (Minikube workaround)

**Note:** Minikube doesn’t support real LoadBalancers, but it simulates them:

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

```bash
kubectl apply -f manifests/loadbalancer.yaml
minikube tunnel
```

---

## 🧹 Cleanup

```bash
kubectl delete -f clusterip.yaml
kubectl delete -f nodeport.yaml
kubectl delete -f loadbalancer.yaml
```

Also, delete deployment if not needed:

```bash
kubectl delete -f deployment.yaml
```

---

## 🧠 Summary

| Type         | Scope               | Use Case                         |
| ------------ | ------------------- | -------------------------------- |
| ClusterIP    | Internal only       | Service-to-service communication |
| NodePort     | External access     | Testing without cloud LB         |
| LoadBalancer | External LB (cloud) | Production workloads             |

---

