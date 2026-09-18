# AWS EC2 Service Discovery — Complete Hands-on Lab

In this lab, we will configure Prometheus to automatically discover EC2 instances using AWS EC2 Service Discovery.

Instead of manually configuring EC2 private IP addresses, Prometheus will query the AWS EC2 API and dynamically discover instances.

---

# Architecture

```text
                         AWS

        ┌─────────────────────────────────┐
        │                                 │
        │  EC2 — Prometheus               │
        │                                 │
        │       Prometheus :9090          │
        │              │                  │
        │              │ EC2 API          │
        │              ▼                  │
        │        AWS EC2 Discovery        │
        │              │                  │
        │       ┌──────┼──────┐            │
        │       ▼      ▼      ▼            │
        │     node-01 node-02 node-03     │
        │      :9100   :9100   :9100       │
        │                                 │
        └─────────────────────────────────┘
```

---

# Learning Objectives

By the end of this lab, you will understand:

* AWS EC2 Service Discovery
* `ec2_sd_configs`
* IAM roles
* IAM policies
* EC2 metadata
* EC2 tags
* `relabel_configs`
* Dynamic EC2 discovery
* Filtering instances
* Using EC2 metadata as Prometheus labels

---

# Prerequisites

You need:

* AWS account
* Two or more EC2 instances
* Prometheus installed on one EC2 instance
* Node Exporter installed on the target EC2 instances

Recommended architecture:

```text
EC2 #1
Prometheus

EC2 #2
Node Exporter

EC2 #3
Node Exporter
```

All instances should be in the same VPC or otherwise have private network connectivity.

---

# Step 1 — Install Node Exporter

On each target EC2 instance:

```bash
sudo useradd \
  --no-create-home \
  --shell /bin/false \
  node_exporter
```

Install the current Node Exporter binary and place it at:

```text
/usr/local/bin/node_exporter
```

Create the systemd service:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter

ExecStart=/usr/local/bin/node_exporter

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Verify:

```bash
sudo systemctl status node_exporter
```

Test locally:

```bash
curl localhost:9100/metrics
```

---

# Step 2 — Configure Security Groups

Prometheus needs to reach Node Exporter.

Allow:

```text
TCP 9100
Source: Prometheus EC2 Security Group
```

Do **not** expose port `9100` publicly unless there is a specific reason.

From the Prometheus EC2 instance:

```bash
curl http://<TARGET_PRIVATE_IP>:9100/metrics
```

You should receive Prometheus metrics.

---

# Step 3 — Create EC2 Tags

We will use tags to control discovery.

Example:

## Prometheus

```text
Name = prometheus
Monitoring = prometheus
```

## Node 01

```text
Name = node-01
Monitoring = enabled
Environment = lab
```

## Node 02

```text
Name = node-02
Monitoring = enabled
Environment = lab
```

The important tag is:

```text
Monitoring = enabled
```

---

# Step 4 — Understand the Static Configuration Problem

The traditional configuration looks like:

```yaml
scrape_configs:

  - job_name: node

    static_configs:

      - targets:
          - 10.0.1.10:9100
          - 10.0.1.11:9100
```

Now imagine `node-01` is terminated.

A replacement instance receives:

```text
10.0.1.25
```

Prometheus still has:

```text
10.0.1.10:9100
```

We have to manually update the configuration.

AWS Service Discovery solves this problem.

---

# Step 5 — Introduce AWS EC2 Service Discovery

Prometheus supports:

```yaml
ec2_sd_configs:
```

Basic example:

```yaml
scrape_configs:

  - job_name: aws-ec2

    ec2_sd_configs:

      - region: eu-central-1
```

Replace:

```text
eu-central-1
```

with your AWS region.

The architecture is now:

```text
Prometheus
     │
     ▼
AWS EC2 API
     │
     ▼
EC2 Instances
```

---

# Step 6 — IAM Authentication

Prometheus needs permission to query the EC2 API.

We need:

```text
Prometheus EC2
       │
       ▼
IAM Role
       │
       ▼
EC2 API
```

We should **not** put permanent AWS access keys inside:

```text
prometheus.yml
```

Instead, attach an IAM role to the Prometheus EC2 instance.

---

# Step 7 — Create the IAM Policy

Create:

```text
iam-policy.json
```

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

The required permission is:

```text
ec2:DescribeInstances
```

---

# Step 8 — Create the IAM Role

Create an IAM role:

```text
prometheus-ec2-role
```

Attach the policy created above.

Then attach the role to the **Prometheus EC2 instance**.

---

# Step 9 — Verify the IAM Role

From the Prometheus EC2 instance:

```bash
aws sts get-caller-identity
```

The result should show the IAM role identity.

If AWS CLI isn't installed:

```bash
sudo apt update
sudo apt install -y awscli
```

---

# Step 10 — Configure Prometheus

Update:

```text
prometheus.yml
```

Start with:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: aws-ec2

    ec2_sd_configs:

      - region: eu-central-1
```

Restart:

```bash
sudo systemctl restart prometheus
```

Check:

```bash
sudo systemctl status prometheus
```

---

# Step 11 — Explore Service Discovery

Open:

```text
Prometheus
   ↓
Status
   ↓
Service Discovery
```

AWS exposes useful metadata such as:

```text
__meta_ec2_instance_id
__meta_ec2_instance_type
__meta_ec2_private_ip
__meta_ec2_private_dns_name
__meta_ec2_tag_Name
__meta_ec2_tag_Monitoring
__meta_ec2_tag_Environment
```

These are discovery labels.

---

# Step 12 — Filter Instances Using Tags

Currently Prometheus may discover many EC2 instances.

We only want:

```text
Monitoring=enabled
```

Add:

```yaml
relabel_configs:

  - source_labels:
      - __meta_ec2_tag_Monitoring

    action: keep

    regex: enabled
```

The flow is:

```text
All EC2 Instances
       │
       ▼
Monitoring=enabled?
       │
   ┌───┴───┐
  YES      NO
   │        │
   ▼        X
 Scrape   Ignore
```

---

# Step 13 — Build the Node Exporter Address

AWS discovery gives Prometheus the EC2 private IP.

We need:

```text
10.0.1.25:9100
```

Add:

```yaml
- source_labels:
    - __meta_ec2_private_ip

  regex: (.+)

  target_label: __address__

  replacement: ${1}:9100
```

Now:

```text
__meta_ec2_private_ip
        │
        ▼
   10.0.1.25
        │
        ▼
__address__
        │
        ▼
10.0.1.25:9100
```

---

# Step 14 — Add Useful Labels

We can turn EC2 tags into Prometheus labels.

Add:

```yaml
- source_labels:
    - __meta_ec2_tag_Name

  target_label: instance_name
```

And:

```yaml
- source_labels:
    - __meta_ec2_tag_Environment

  target_label: environment
```

Now a metric can contain:

```text
instance_name="node-01"
environment="lab"
```

instead of relying only on:

```text
10.0.1.25:9100
```

---

# Step 15 — Complete Configuration

Your final configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: aws-ec2

    ec2_sd_configs:

      - region: eu-central-1

    relabel_configs:

      # Only discover EC2 instances
      # with Monitoring=enabled

      - source_labels:
          - __meta_ec2_tag_Monitoring

        action: keep

        regex: enabled


      # Convert private IP into
      # Node Exporter address

      - source_labels:
          - __meta_ec2_private_ip

        regex: (.+)

        target_label: __address__

        replacement: ${1}:9100


      # Add EC2 Name tag

      - source_labels:
          - __meta_ec2_tag_Name

        target_label: instance_name


      # Add Environment tag

      - source_labels:
          - __meta_ec2_tag_Environment

        target_label: environment
```

---

# Step 16 — Verify Targets

Open:

```text
Status
   ↓
Targets
```

You should see:

```text
aws-ec2

node-01    UP
node-02    UP
```

Query:

```promql
up{job="aws-ec2"}
```

---

# Step 17 — Query Using the New Labels

Because we created:

```text
instance_name
environment
```

we can query:

```promql
up{environment="lab"}
```

Or:

```promql
up{instance_name="node-01"}
```

---

# Step 18 — Dynamic Discovery Demo

This is the main demo.

Launch another EC2 instance:

```text
node-03
```

Install Node Exporter.

Add:

```text
Name = node-03
Monitoring = enabled
Environment = lab
```

Do **not** modify:

```text
prometheus.yml
```

Wait for the discovery cycle.

Go to:

```text
Prometheus
→ Status
→ Targets
```

You should see:

```text
node-01    UP
node-02    UP
node-03    UP
```

No Prometheus configuration change was required.

---

# Step 19 — Termination Demo

Terminate:

```text
node-02
```

AWS removes the instance.

Prometheus periodically refreshes its discovered EC2 targets.

Eventually:

```text
node-02
```

disappears from the discovered targets.

The flow:

```text
EC2 terminated
      ↓
AWS EC2 API
      ↓
Service Discovery
      ↓
Prometheus
      ↓
Target removed
```

---

# Step 20 — Filter by Environment

Suppose we have:

```text
Environment=production
Environment=staging
Environment=lab
```

We can filter:

```yaml
- source_labels:
    - __meta_ec2_tag_Environment

  action: keep

  regex: production
```

Now only production instances are discovered.

---

# Step 21 — Grafana Integration

The discovered targets can now be visualized in Grafana.

Example CPU query:

```promql
100 * (
  1 -
  avg by(instance_name) (
    rate(
      node_cpu_seconds_total{
        job="aws-ec2",
        mode="idle"
      }[5m]
    )
  )
)
```

The dashboard can display:

```text
node-01    23%
node-02    41%
node-03    17%
```

instead of:

```text
10.0.1.25:9100
10.0.1.26:9100
10.0.1.27:9100
```

---

# Final Architecture

```text
                       AWS

                  EC2 API
                     ▲
                     │
              IAM Instance Role
                     │
                     │
              ┌──────┴──────┐
              │ Prometheus  │
              │    :9090    │
              └──────┬──────┘
                     │
               EC2 Service
                Discovery
                     │
             ┌───────┼────────┐
             ▼       ▼        ▼
          node-01 node-02  node-03
           :9100   :9100    :9100
```

---

# Key Takeaway

The important concept is:

```text
AWS EC2 API
      ↓
EC2 Service Discovery
      ↓
EC2 Metadata
      ↓
Relabeling
      ↓
Scrape Targets
      ↓
Prometheus
```

Instead of manually maintaining:

```text
10.0.1.10:9100
10.0.1.11:9100
10.0.1.12:9100
```

we tell Prometheus:

```text
"Discover EC2 instances with Monitoring=enabled."
```

This allows monitoring to follow dynamically changing AWS infrastructure.
