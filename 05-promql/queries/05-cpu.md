# CPU

CPU utilization:

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
