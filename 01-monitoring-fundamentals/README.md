# Module 01 — Monitoring Fundamentals

## Objectives

- Understand monitoring
- Compare metrics, logs, and traces
- Understand why Prometheus is widely used in Cloud Native environments
- Understand Prometheus architecture
- Understand Prometheus, exporters, service discovery, Alertmanager, and Grafana

---

## 1. Introduction to Monitoring & Observability

### What is Monitoring?
Monitoring is the process of collecting, analyzing, and using information to track a system's health, performance, and availability. It answers the fundamental question: **"Is the system working?"**

### The Shift to Observability
In dynamic microservice and containerized environments, systems fail in complex, unpredictable ways. **Observability** measures how well internal states can be inferred from external outputs (telemetry). It answers the underlying diagnostic question: **"Why is the system failing?"**

## 2. Metrics vs. Logs vs. Traces

To achieve complete observability, systems rely on three distinct telemetry data types:

| Telemetry Type | Definition | Primary Use Case | Example | Pros & Cons |
| :--- | :--- | :--- | :--- | :--- |
| **Metrics** | Numeric values aggregated over time intervals representing system state. | Alerting, real-time dashboards, capacity planning. | `http_requests_total = 1452` | **+** Fast to query, low storage footprint.<br>**-** Lacks individual transaction context. |
| **Logs** | Time-stamped text records of discrete events emitted by applications. | Debugging specific failures, auditing. | `[ERROR] User 42 failed DB connection` | **+** High detail and context.<br>**-** High storage cost, slow at scale. |
| **Traces** | End-to-end request journeys tracked across multiple distributed services. | Profiling latency bottlenecks, dependency mapping. | `API Gateway (10ms) -> Auth (5ms) -> DB (120ms)` | **+** Visualizes request paths.<br>**-** Complex instrumentation overhead. |

### Diagnostic Workflow
1. **Metric** triggers an alert: *HTTP 500 error rate spiked to 12% on `payments-service`*.
2. **Trace** pinpoints the bottleneck: *Request failed during the call from `payments-service` to `db-cluster`*.
3. **Log** identifies the root cause: *`FATAL: Connection pool exhausted at 10:14:02 UTC`*.

---

## 3. Why Prometheus in Cloud-Native Environments?

Traditional monitoring solutions struggle with ephemeral infrastructure like Kubernetes or cloud-native container runtimes. Prometheus was built specifically for these environments:

* **Pull-Based Telemetry:** Prometheus actively scrapes targets over HTTP endpoints (`/metrics`). This prevents application targets from needing to know server IPs or overloading ingestion pipelines during traffic spikes.
* **Dimensional Data Model:** Metrics are identified by name and arbitrary key-value pairs called labels (`http_requests_total{method="POST", status="500"}`), allowing high-granularity aggregation.
* **Service Discovery:** Native integrations with Kubernetes, AWS, GCP, and Consul dynamically discover and monitor infrastructure components as they scale up or down.
* **Independent Architecture:** Each Prometheus server is an autonomous, single-node instance that stores data locally, eliminating dependencies on external distributed databases during outages.
* **PromQL:** A powerful functional query language optimized for real-time mathematical operations over time-series data.

---

## 4. Prometheus Architecture
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9f3d104d-29c4-489a-8427-e881057a11f6" />

### Core Components
* **Prometheus Server:** Scrapes endpoints, evaluates alert rules, and persists data to a local Time Series Database (TSDB).
* **Pushgateway:** Accepts metrics pushed from short-lived batch jobs that do not live long enough to be scraped directly.
* **Alertmanager:** Deduplicates, groups, and routes triggered alerts to external systems (Slack, PagerDuty, Email).
* **Grafana:** Connects to Prometheus via PromQL to render visual operational dashboards.

## 5. Ecosystem Components
* **Exporters:** Translation proxies that expose non-Prometheus systems (OS, databases, message queues) as readable metrics endpoints.
  * *Node Exporter:* Hardware and kernel-level Linux OS metrics.
  * *cAdvisor:* Container resource utilization metrics.
* **Service Discovery:** Mechanisms that continuously update scrape target endpoints without manual static updates to configuration files.
* **Alertmanager:** Handles notification routing, suppressing cascading failure noise (inhibition), and silencing during maintenance.
* **Grafana:** Visualization interface used for charting long-term metrics and monitoring system state.
---
