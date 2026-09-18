# Kubernetes Service Discovery — Complete Hands-on Lab

In this lab, we will connect Prometheus to a Kubernetes cluster and use Kubernetes Service Discovery to automatically discover Pods.

## What We Will Build

```text
                    Kind Cluster

             ┌───────────────────────┐
             │   Kubernetes API      │
             │                       │
             │          ▲            │
             │          │            │
             │         RBAC          │
             │          │            │
             │          │            │
             │   ┌──────┴──────┐     │
             │   │ Prometheus  │     │
             │   │    :9090    │     │
             │   └──────┬─────┘     │
             │          │            │
             │    Kubernetes SD      │
             │          │            │
             │     ┌────┼────┐       │
             │     ▼    ▼    ▼       │
             │   Pod A Pod B Pod C   │
             │                       │
             └───────────────────────┘
```

---

# Learning Objectives

By the end of this lab, you will understand:

* How to create a Kubernetes cluster with Kind
* How Prometheus communicates with the Kubernetes API
* Kubernetes Service Discovery
* `kubernetes_sd_configs`
* Kubernetes RBAC
* ServiceAccounts
* `ClusterRole`
* `ClusterRoleBinding`
* Discovery metadata
* `relabel_configs`
* Dynamic Pod discovery

---

# Prerequisites

Install:

```bash
docker
kubectl
kind
```

Verify:

```bash
docker --version
kubectl version --client
kind version
```

---

# Step 1 — Create the Kind Cluster

Create a cluster:

```bash
kind create cluster --name monitoring-lab
```

Verify:

```bash
kubectl cluster-info
```

Check the node:

```bash
kubectl get nodes
```

Expected:

```text
NAME                           STATUS   ROLES
monitoring-lab-control-plane   Ready    control-plane
```

---

# Step 2 — Explore Kubernetes

Run:

```bash
kubectl get all -A
```

Then:

```bash
kubectl get namespaces
```

And:

```bash
kubectl get pods -A -o wide
```

## Important Concept

Kubernetes stores information about its resources in the:

```text
Kubernetes API
```

Prometheus will use this API to discover monitoring targets.

---

# Step 3 — Create a Namespace

```bash
kubectl create namespace monitoring-demo
```

Verify:

```bash
kubectl get namespaces
```

---

# Step 4 — Deploy the Demo Application

Create:

```text
demo-app.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: monitoring-demo

spec:
  replicas: 3

  selector:
    matchLabels:
      app: demo-app

  template:
    metadata:
      labels:
        app: demo-app
        monitoring: enabled

    spec:
      containers:

        - name: nginx
          image: nginx:1.27

          ports:
            - name: http
              containerPort: 80
```

Apply:

```bash
kubectl apply -f demo-app.yaml
```

Check:

```bash
kubectl get pods \
  -n monitoring-demo \
  -o wide
```

You should see three Pods.

---

# Step 5 — Understand the Problem

Suppose we manually configure:

```yaml
static_configs:
  - targets:
      - 10.244.0.5:80
      - 10.244.0.6:80
      - 10.244.0.7:80
```

Now delete one Pod:

```bash
kubectl delete pod \
  <POD_NAME> \
  -n monitoring-demo
```

Kubernetes creates a replacement.

Check:

```bash
kubectl get pods \
  -n monitoring-demo \
  -o wide
```

The new Pod will probably have a different IP.

This demonstrates why static configuration isn't ideal for Kubernetes.

---

# Step 6 — Kubernetes Service Discovery

Prometheus supports:

```yaml
kubernetes_sd_configs:
```

Example:

```yaml
scrape_configs:

  - job_name: kubernetes-pods

    kubernetes_sd_configs:

      - role: pod
```

The:

```text
role: pod
```

tells Prometheus:

> Discover Kubernetes Pods.

Other available roles include:

```text
pod
service
node
endpoints
endpointslice
```

---

# Step 7 — Kubernetes RBAC

Prometheus needs permission to query the Kubernetes API.

Create:

```text
rbac.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount

metadata:
  name: prometheus
  namespace: monitoring-demo

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole

metadata:
  name: prometheus

rules:

  - apiGroups:
      - ""

    resources:
      - pods
      - nodes
      - services
      - endpoints

    verbs:
      - get
      - list
      - watch

  - apiGroups:
      - discovery.k8s.io

    resources:
      - endpointslices

    verbs:
      - get
      - list
      - watch

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding

metadata:
  name: prometheus

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus

subjects:

  - kind: ServiceAccount
    name: prometheus
    namespace: monitoring-demo
```

Apply:

```bash
kubectl apply -f rbac.yaml
```

---

# Step 8 — Understand the RBAC Permissions

We gave Prometheus:

```text
get
list
watch
```

Why?

### `get`

Read resources.

### `list`

List resources.

### `watch`

Watch for changes.

The `watch` permission is particularly important for dynamic environments.

---

# Step 9 — Verify RBAC

Run:

```bash
kubectl auth can-i \
  list pods \
  --as=system:serviceaccount:monitoring-demo:prometheus
```

Expected:

```text
yes
```

Try:

```bash
kubectl auth can-i \
  delete pods \
  --as=system:serviceaccount:monitoring-demo:prometheus
```

Expected:

```text
no
```

Prometheus only needs to discover resources. It doesn't need permission to modify them.

---

# Step 10 — Deploy Prometheus

For this lab, Prometheus will run **inside Kubernetes**.

Create:

```text
prometheus-config.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: prometheus-config
  namespace: monitoring-demo

data:

  prometheus.yml: |

    global:
      scrape_interval: 15s

    scrape_configs:

      - job_name: prometheus

        static_configs:
          - targets:
              - localhost:9090

      - job_name: kubernetes-pods

        kubernetes_sd_configs:

          - role: pod
```

Apply:

```bash
kubectl apply \
  -f prometheus-config.yaml
```

---

# Step 11 — Prometheus Deployment

Create:

```text
prometheus-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: prometheus
  namespace: monitoring-demo

spec:
  replicas: 1

  selector:
    matchLabels:
      app: prometheus

  template:

    metadata:
      labels:
        app: prometheus

    spec:

      serviceAccountName: prometheus

      containers:

        - name: prometheus

          image: prom/prometheus:latest

          args:
            - --config.file=/etc/prometheus/prometheus.yml

          ports:
            - containerPort: 9090

          volumeMounts:

            - name: config
              mountPath: /etc/prometheus

      volumes:

        - name: config

          configMap:
            name: prometheus-config
```

Apply:

```bash
kubectl apply \
  -f prometheus-deployment.yaml
```

Check:

```bash
kubectl get pods \
  -n monitoring-demo
```

---

# Step 12 — Access Prometheus

Use port forwarding:

```bash
kubectl port-forward \
  deployment/prometheus \
  9090:9090 \
  -n monitoring-demo
```

Open:

```text
http://localhost:9090
```

---

# Step 13 — Explore Kubernetes Service Discovery

In Prometheus:

```text
Status
  ↓
Service Discovery
```

You will see Kubernetes discovery information.

Important metadata includes:

```text
__meta_kubernetes_namespace
__meta_kubernetes_pod_name
__meta_kubernetes_pod_ip
__meta_kubernetes_pod_node_name
__meta_kubernetes_pod_container_name
```

These are discovery labels.

---

# Step 14 — Introduce Relabeling

Discovery can find many resources.

We don't necessarily want to scrape all of them.

Our Pods have:

```yaml
monitoring: enabled
```

We can filter using:

```yaml
relabel_configs:
```

Add:

```yaml
relabel_configs:

  - source_labels:
      - __meta_kubernetes_pod_label_monitoring

    action: keep

    regex: enabled
```

The logic is:

```text
All discovered Pods
        │
        ▼
monitoring=enabled?
        │
    ┌───┴───┐
   YES      NO
    │        │
    ▼        X
 scrape    ignore
```

---

# Step 15 — Filter by Namespace

We can also restrict discovery to:

```text
monitoring-demo
```

Add:

```yaml
- source_labels:
    - __meta_kubernetes_namespace

  action: keep

  regex: monitoring-demo
```

Now Prometheus only keeps targets from this namespace.

---

# Step 16 — Dynamic Scaling Demo

Start with:

```bash
kubectl get pods \
  -n monitoring-demo
```

We have:

```text
3 Pods
```

Scale:

```bash
kubectl scale deployment demo-app \
  --replicas=5 \
  -n monitoring-demo
```

Check:

```bash
kubectl get pods \
  -n monitoring-demo
```

Now we have:

```text
5 Pods
```

Go to:

```text
Prometheus
→ Status
→ Service Discovery
```

The newly created Pods are discovered automatically.

**No Prometheus configuration change was required.**

---

# Step 17 — Pod Replacement Demo

Delete a Pod:

```bash
kubectl delete pod \
  <POD_NAME> \
  -n monitoring-demo
```

Watch:

```bash
kubectl get pods \
  -n monitoring-demo \
  -w
```

The flow is:

```text
Pod deleted
     ↓
ReplicaSet detects missing replica
     ↓
New Pod created
     ↓
New Pod IP
     ↓
Kubernetes API updated
     ↓
Prometheus discovers new Pod
```

This demonstrates the real value of Kubernetes Service Discovery.

---

# Step 18 — Service Discovery Roles

Prometheus supports multiple Kubernetes discovery roles.

### Pod

```yaml
role: pod
```

Discovers Pods.

### Service

```yaml
role: service
```

Discovers Services.

### Node

```yaml
role: node
```

Discovers Kubernetes nodes.

### Endpoints

```yaml
role: endpoints
```

Discovers endpoints associated with Services.

### EndpointSlice

```yaml
role: endpointslice
```

Discovers EndpointSlices.

---

# Final Architecture

```text
                  Kubernetes API
                        ▲
                        │
                       RBAC
                        │
                        │
                 ┌──────┴──────┐
                 │ Prometheus  │
                 │    :9090    │
                 └──────┬──────┘
                        │
                Kubernetes SD
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           Pod A      Pod B      Pod C
          10.244.x   10.244.x   10.244.x
```

## Key Takeaway

The important concept is:

```text
Kubernetes API
      ↓
Service Discovery
      ↓
Discovery Metadata
      ↓
Relabeling
      ↓
Scrape Targets
      ↓
Prometheus
```

Kubernetes can create, delete, and replace Pods continuously.

Prometheus Service Discovery allows monitoring to follow those changes automatically.
