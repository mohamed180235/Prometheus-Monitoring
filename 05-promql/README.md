# Module 05 — PromQL

PromQL is the query language used by Prometheus.

## 1. Metric Selection

```promql
up
```

```promql
node_memory_MemAvailable_bytes
```

## 2. Labels

```promql
node_cpu_seconds_total{mode="idle"}
```

Operators:

```promql
mode="idle"
mode!="idle"
mode=~"user|system"
mode!~"idle|iowait"
```

## 3. Instant vs Range

Instant vector:

```promql
node_cpu_seconds_total
```

Range vector:

```promql
node_cpu_seconds_total[5m]
```

## 4. Counter and rate()

```promql
rate(node_cpu_seconds_total[5m])
```

## 5. Memory Utilization

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

## 6. CPU Utilization

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

## 7. Aggregation

```promql
sum(rate(node_cpu_seconds_total[5m]))
```

```promql
avg by(instance) (
  rate(node_cpu_seconds_total[5m])
)
```

```promql
count by(job)(up)
```

## 8. increase()

```promql
increase(http_requests_total[1h])
```

## 9. Top Containers by CPU

```promql
topk(
  5,
  sum by(name) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

## 10. Top Containers by Memory

```promql
topk(
  5,
  sum by(name) (
    container_memory_usage_bytes
  )
)
```

## 11. Histogram P95

```promql
histogram_quantile(
  0.95,
  sum by(le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

See the `queries/` directory for a larger cheat sheet.
