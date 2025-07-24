
# 📘 **Day 7: Resource Requests & Limits in Kubernetes**

---

## ✅ What You’ll Learn

| Topic                    | Description                                  |
| ------------------------ | -------------------------------------------- |
| ⚙️ **Resource Requests** | Minimum CPU/memory a container is guaranteed |
| 🔒 **Resource Limits**   | Maximum CPU/memory a container can use       |
| 🧪 **Hands-on**          | Apply limits to Pods and simulate load       |
| 🧹 **Cleanup**           | Remove pods and related resources            |

---

## 🧠 Why Are Requests & Limits Important?

1. **Avoid cluster overloads** – prevents one pod from hogging all resources
2. **Enable fair scheduling** – scheduler uses **requests** to place pods
3. **Prevent OOM kills** – with memory limits, you can catch misbehaving apps
4. **Billing/cost control** – avoid resource waste in cloud deployments

---

## ⚙️ Key Concepts

| Resource Type | Unit             | Description                      |
| ------------- | ---------------- | -------------------------------- |
| CPU           | `millicores (m)` | 500m = 0.5 core                  |
| Memory        | `Mi` or `Gi`     | Megabytes/Gigabytes              |
| `requests`    | Guaranteed min   | Scheduler uses this for planning |
| `limits`      | Absolute max     | Container can’t go beyond this   |

---

## 📄 Sample Pod with Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "while true; do echo Working; sleep 5; done"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

---

### 🧪 Apply It

```bash
kubectl apply -f resource-demo.yaml
kubectl describe pod resource-demo
```

---

### 🧪 Simulate High CPU Load (optional)

```bash
kubectl exec -it resource-demo -- sh
yes > /dev/null
```

> Watch the CPU usage:

```bash
kubectl top pod resource-demo
```

---

## 🧠 What Happens If…

| Scenario                                 | Result                                 |
| ---------------------------------------- | -------------------------------------- |
| Pod exceeds **limit**                    | Throttled (CPU), or OOMKilled (Memory) |
| Pod exceeds **request**, but under limit | Allowed, if resources available        |
| Node lacks **request** resources         | Pod won’t be scheduled                 |

---

## 🎯 Namespace Defaults (LimitRange)

You can define **default limits** for all pods in a namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limit
  namespace: dev
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "128Mi"
    defaultRequest:
      cpu: "250m"
      memory: "64Mi"
    type: Container
```

> 🧪 Apply:

```bash
kubectl create ns dev
kubectl apply -f limit-range.yaml
```

Now any pod created in `dev` namespace gets default limits!

---

## 🧹 Cleanup

```bash
kubectl delete pod resource-demo
kubectl delete ns dev
```

---

## 📊 Monitoring Resource Usage

```bash
kubectl top pod
kubectl top node
```

> Requires Metrics Server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## ✅ Summary

| Concept    | Summary                                             |
| ---------- | --------------------------------------------------- |
| `requests` | Min resources guaranteed                            |
| `limits`   | Max allowed resource usage                          |
| Use cases  | Scheduling, budgeting, QoS, stability               |
| Tools      | `kubectl describe`, `kubectl top`, `metrics-server` |

---


