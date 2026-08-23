# Histograms

Bucket rate:

```promql
sum by(le)(
  rate(http_request_duration_seconds_bucket[5m])
)
```

P95:

```promql
histogram_quantile(
  0.95,
  sum by(le)(
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

Average latency:

```promql
rate(http_request_duration_seconds_sum[5m])
/
rate(http_request_duration_seconds_count[5m])
```
