# Aggregation

```promql
sum(rate(node_cpu_seconds_total[5m]))
```

```promql
sum by(instance)(
  rate(node_cpu_seconds_total[5m])
)
```

```promql
avg by(instance)(
  rate(node_cpu_seconds_total[5m])
)
```

```promql
count by(job)(up)
```
