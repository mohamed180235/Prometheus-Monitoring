# Module 06 — Grafana

Grafana is the visualization layer for the Prometheus metrics.

## Install

Install Grafana on EC2 #1 using the official Grafana Ubuntu/Debian installation instructions.

Start:

```bash
sudo systemctl enable --now grafana-server
```

Open:

```text
http://<PROMETHEUS_PUBLIC_IP>:3000
```

## Add Prometheus

Create a Prometheus data source:

```text
http://localhost:9090
```

Click **Save & Test**.

## First Panel

Query:

```promql
up
```

Use a Stat visualization.

## CPU Panel

```promql
100 * (
  1 -
  avg by(instance) (
    rate(
      node_cpu_seconds_total{mode="idle"}[5m]
    )
  )
)
```

Use Time series and set the unit to Percent.

## Memory Panel

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

## Disk Gauge

```promql
100 * (
  1 -
  node_filesystem_avail_bytes{mountpoint="/"}
  /
  node_filesystem_size_bytes{mountpoint="/"}
)
```

## Container CPU

```promql
topk(
  5,
  sum by(name)(
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

## Variables

Create an `instance` variable using:

```promql
label_values(node_cpu_seconds_total, instance)
```

Then use:

```promql
node_cpu_seconds_total{instance="$instance"}
```

## Explore vs Dashboard

**Explore** is for investigation and query development.

**Dashboard** is for repeatable monitoring.
