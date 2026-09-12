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

```text
Application / Host
       │
       ▼
   Exporter
       │ /metrics
       ▼
 Prometheus
       │
       ├── PromQL
       │
       ▼
   Grafana

Prometheus alerts
       │
       ▼
 Alertmanager
```

## Metric Types Demo

Use the following examples:

### Counter

```promql
node_cpu_seconds_total
```

### Gauge

```promql
node_memory_MemAvailable_bytes
```

### Histogram

Typical application metric:

```text
http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count
```

### Summary

Typical application metric:

```text
http_request_duration_seconds{quantile="0.95"}
```

## Key Exercise

Explain why `rate()` is normally used with counters but not directly with gauges.
