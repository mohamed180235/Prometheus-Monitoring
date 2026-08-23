# rate() and increase()

Counter:

```promql
node_cpu_seconds_total
```

Rate:

```promql
rate(node_cpu_seconds_total[5m])
```

Increase:

```promql
increase(http_requests_total[1h])
```

Remember:

- `rate()` = average per-second increase
- `increase()` = total increase over the selected range
