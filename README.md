# 🧠 What is Helm?
Helm is like a package manager for Kubernetes — similar to how:
- apt/yum manage packages in Linux
- npm manages packages in Node.js
In Kubernetes, you usually deploy applications using many YAML files (`deployment.yaml`, `service.yaml`, `configmap.yaml`, etc).
That can get messy — so Helm helps by packaging all these YAMLs into one reusable unit called a Helm Chart.
---
## 📦 What is a Helm Chart?
A Helm Chart is a folder containing all the Kubernetes manifests (YAML files) + a configuration file (values.yaml) that lets you customize them easily.
Example structure - 
myapp/
 ├── Chart.yaml         # Info about the chart
 ├── values.yaml        # Default configuration values
 ├── templates/         # Actual Kubernetes YAML templates
 │    ├── deployment.yaml
 │    ├── service.yaml
 │    └── ingress.yaml

---

## 🚀 Example Without Helm (Traditional Way)
You might have to run:
```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```
And if you want to change something (like replica count or image version),
you must manually edit multiple YAML files. 😩

---

## 🚀 Example With Helm
With Helm, you can deploy all those YAMLs together as a single release:
```sh
helm install myapp ./myapp
```
And to update (say, change image tag or replicas):
```sh
helm upgrade myapp ./myapp --set image.tag=v2
```
Helm automatically detects changes and applies updates in order.
No need to manually handle YAMLs or kubectl apply each one.

---
## 🌍 Real-World Example
Let’s say you want to install NGINX Ingress Controller.

Without Helm → You’d need to apply ~10 YAML files manually.
With Helm → Just run:
```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-ingress ingress-nginx/ingress-nginx
```
# 📦 Helm Chart Structure Explained

A Helm chart helps package and deploy Kubernetes applications easily. Below is a breakdown of each file and folder in a typical Helm chart with examples.

```
myapp/
 ├── Chart.yaml         # Info about the chart
 ├── values.yaml        # Default configuration values
 ├── templates/         # Actual Kubernetes YAML templates
 │    ├── deployment.yaml
 │    ├── service.yaml
 │    └── ingress.yaml
```

---

## 🧾 1. Chart.yaml

This file contains **metadata about the chart** — like name, version, and description.

**Example:**

```yaml
apiVersion: v2
name: myapp
description: A simple NGINX web app chart
type: application
version: 0.1.0
appVersion: "1.0"
```

**Explanation:**

* `name`: Chart name
* `version`: Chart version (used for updates)
* `appVersion`: Version of the app being deployed
* `type`: Usually `application`

---

## ⚙️ 2. values.yaml

Holds **default configuration values** used by templates. You can override them during installation or upgrade.

**Example:**

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: latest

service:
  type: ClusterIP
  port: 80
```

**Explanation:**

* Define parameters like replicas, image, and service type.
* Templates refer to these using `{{ .Values.variableName }}`.

---

## 🧩 3. templates/

Contains **Kubernetes YAML templates** that define your app components. Each template can use values from `values.yaml`.

### 🏗️ deployment.yaml

Defines **Pods and ReplicaSets**.

**Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

**Explanation:**

* Dynamically uses image name, tag, and replicas from `values.yaml`.

---

### 🌐 service.yaml

Defines **how the app is exposed** inside or outside the cluster.

**Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Chart.Name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ .Chart.Name }}
```

**Explanation:**

* Type can be `ClusterIP`, `NodePort`, or `LoadBalancer`.
* Uses values from `values.yaml` for flexibility.

---

### 🚪 ingress.yaml

(Optional) Defines **external access** using domain names or paths.

**Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Chart.Name }}
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ .Chart.Name }}
                port:
                  number: {{ .Values.service.port }}
```

**Explanation:**

* Routes traffic from `myapp.local` to the Kubernetes Service.

---

## ✅ Summary

| File              | Purpose                      | Example Use                 |
| ----------------- | ---------------------------- | --------------------------- |
| `Chart.yaml`      | Chart metadata               | Name, version, appVersion   |
| `values.yaml`     | Default configuration values | Image, replicas, ports      |
| `deployment.yaml` | Defines app Pods             | Uses image + replicas       |
| `service.yaml`    | Exposes Pods                 | Internal or external access |
| `ingress.yaml`    | Routes traffic               | Maps hostname to service    |

---

## 🚀 Usage

Render templates locally for preview:

```bash
helm template ./myapp
```

Install chart to Kubernetes:

```bash
helm install myapp ./myapp
```

Upgrade or rollback:

```bash
helm upgrade myapp ./myapp
helm rollback myapp 1
```
