# Module 03 — Configuration & Service Discovery

This module explains how Prometheus discovers the targets it needs to monitor.

We will start with simple static configuration and gradually move to dynamic service discovery.

## Learning Objectives

By the end of this module, you will understand:

* Prometheus scrape configuration
* Jobs and targets
* Static target discovery
* File-based service discovery
* Kubernetes service discovery
* AWS EC2 service discovery
* `relabel_configs`
* Discovery metadata
* Dynamic target changes

---

# Service Discovery

Prometheus needs to know:

> **Which endpoints should I scrape?**

The simplest approach is to configure them manually:

```yaml
scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - 10.0.1.10:9100
          - 10.0.1.11:9100
```

This works, but it doesn't scale well when infrastructure changes frequently.

For dynamic environments, Prometheus can discover targets automatically.

Examples:

```text
Kubernetes
AWS EC2
Consul
File-based discovery
DNS
Cloud providers
```

The general architecture is:

```text
Discovery System
       │
       ▼
Prometheus
       │
       ▼
Discovery Metadata
       │
       ▼
Relabeling
       │
       ▼
Scrape Targets
```

---

# Demo 1 — Static Service Discovery

Directory:

```text
static/
└── prometheus.yml
```

Example:

```yaml
scrape_configs:

  - job_name: node

    static_configs:
      - targets:
          - 10.0.1.10:9100
          - 10.0.1.11:9100
```

### Problem

If an instance is replaced:

```text
Old EC2
10.0.1.10
    ↓
terminated

New EC2
10.0.1.25
```

Prometheus still has:

```text
10.0.1.10:9100
```

The configuration must be updated manually.

---

# Demo 2 — File-Based Service Discovery

Directory:

```text
file-sd/
├── prometheus.yml
└── targets/
    └── nodes.json
```

Prometheus can read targets from a file:

```yaml
scrape_configs:

  - job_name: node

    file_sd_configs:
      - files:
          - /etc/prometheus/targets/*.json
```

Example:

```json
[
  {
    "targets": [
      "10.0.1.10:9100"
    ],
    "labels": {
      "environment": "lab"
    }
  }
]
```

The target list can be changed without modifying the main Prometheus configuration.

---

# Demo 3 — Kubernetes Service Discovery

Directory:

```text
kubernetes-sd/
```

We will:

1. Create a Kubernetes cluster using `kind`
2. Deploy a sample application
3. Install Prometheus
4. Create a ServiceAccount
5. Configure RBAC
6. Connect Prometheus to the Kubernetes API
7. Discover Pods dynamically
8. Use relabeling
9. Scale the application
10. Delete and replace Pods
11. Observe Prometheus automatically update its targets

See:

```text
kubernetes-sd/README.md
```

---

# Demo 4 — AWS EC2 Service Discovery

Directory:

```text
aws-ec2-sd/
```

We will:

1. Create EC2 instances
2. Install Node Exporter
3. Create EC2 tags
4. Create an IAM role
5. Give Prometheus permission to describe EC2 instances
6. Configure `ec2_sd_configs`
7. Discover EC2 instances
8. Filter instances using tags
9. Use EC2 metadata as Prometheus labels
10. Launch a new EC2 instance
11. Terminate an EC2 instance
12. Observe dynamic discovery

See:

```text
aws-ec2-sd/README.md
```

---

# Important Concept — Relabeling

Service discovery doesn't automatically mean that every discovered resource should be scraped.

Prometheus can filter and modify discovered targets using:

```yaml
relabel_configs:
```

For example:

```yaml
relabel_configs:

  - source_labels:
      - __meta_ec2_tag_Monitoring

    action: keep

    regex: enabled
```

This means:

```text
EC2 Instances
      │
      ▼
Monitoring=enabled?
      │
   ┌──┴──┐
  YES    NO
   │      │
   ▼      X
 Scrape  Ignore
```

---

# Important Discovery Labels

Different service discovery mechanisms expose different metadata.

## Kubernetes

```text
__meta_kubernetes_namespace
__meta_kubernetes_pod_name
__meta_kubernetes_pod_ip
__meta_kubernetes_pod_node_name
__meta_kubernetes_pod_container_name
```

## AWS EC2

```text
__meta_ec2_instance_id
__meta_ec2_instance_type
__meta_ec2_private_ip
__meta_ec2_private_dns_name
__meta_ec2_tag_Name
__meta_ec2_tag_Environment
```

These labels can be used by `relabel_configs`.

---

# Exercises

### Exercise 1

Create two static Prometheus targets.

### Exercise 2

Move the targets to File SD.

### Exercise 3

Create a Kind cluster and discover Pods using Kubernetes SD.

### Exercise 4

Configure Prometheus to discover only Pods with:

```text
monitoring=enabled
```

### Exercise 5

Configure AWS EC2 SD.

Only discover instances with:

```text
Monitoring=enabled
```

### Exercise 6

Add an `environment` label based on the EC2 tag.

### Exercise 7

Launch a new EC2 instance and verify that Prometheus discovers it without changing `prometheus.yml`.

---

# Key Takeaway

Static discovery requires us to tell Prometheus:

```text
"Here are my targets."
```

Dynamic discovery allows us to tell Prometheus:

```text
"Here is where you can find my targets."
```

This becomes extremely important in cloud-native environments where infrastructure changes continuously.
