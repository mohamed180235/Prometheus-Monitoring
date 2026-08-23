# Labels

```promql
node_cpu_seconds_total{mode="idle"}
```

```promql
node_cpu_seconds_total{mode!="idle"}
```

```promql
node_cpu_seconds_total{mode=~"user|system"}
```

```promql
node_cpu_seconds_total{mode!~"idle|iowait"}
```
