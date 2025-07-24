
## 🧠 What is Helm?

**Helm** is the **package manager for Kubernetes**.

Think of it like:

* `apt` for Ubuntu
* `yum` for CentOS
* `pip` for Python
* 🧢 But for Kubernetes YAML!

Helm bundles Kubernetes manifests into **charts** that are:

* Templated (dynamic & reusable)
* Configurable (values.yaml)
* Version-controlled (stored in chart repositories)

---

## 🔧 Step 1: Install Helm (Windows/macOS/Linux)

### 🪟 On Windows (via Chocolatey):

```bash
choco install kubernetes-helm
```

### 🍎 On macOS (via Homebrew):

```bash
brew install helm
```

### 🐧 On Linux (script):

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Verify installation:

```bash
helm version
```

---

## 📦 Step 2: Add a Public Helm Chart Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Check available charts:

```bash
helm search repo bitnami
```

---

## 🚀 Step 3: Install NGINX Using Helm

```bash
helm install my-nginx bitnami/nginx
```

See the deployed resources:

```bash
kubectl get all
```

Helm keeps track of deployments called **releases**. You can check with:

```bash
helm list
```

---

## 🧼 Step 4: Uninstall the Release

```bash
helm uninstall my-nginx
```

---

## 🧰 Step 5: Create a Custom Helm Chart

```bash
helm create demo-chart
cd demo-chart
```

This creates a structure like:

```
demo-chart/
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
├── values.yaml
├── Chart.yaml
```

Modify these as needed:

* `Chart.yaml`: metadata
* `values.yaml`: default config
* `templates/`: the actual YAML files using Go templating

---

## 🧪 Step 6: Install Your Custom Chart

```bash
helm install my-demo ./demo-chart
```

To upgrade:

```bash
helm upgrade my-demo ./demo-chart
```

To uninstall:

```bash
helm uninstall my-demo
```

---

## 📘 Summary

| Concept  | Meaning                          |
| -------- | -------------------------------- |
| Helm     | K8s package manager              |
| Chart    | Collection of templates + values |
| Template | YAML with dynamic values         |
| Values   | Configurable inputs              |
| Release  | Helm-managed deployment instance |

---



