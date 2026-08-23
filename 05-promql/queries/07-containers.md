# Containers

CPU:

```promql
sum by(name)(
  rate(container_cpu_usage_seconds_total[5m])
)
```

Top 5 CPU:

```promql
topk(
  5,
  sum by(name)(
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

Memory:

```promql
topk(
  5,
  sum by(name)(
    container_memory_usage_bytes
  )
)
```

Network receive:

```promql
sum by(name)(
  rate(container_network_receive_bytes_total[5m])
)
```
