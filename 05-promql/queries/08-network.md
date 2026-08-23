# Network

Host receive:

```promql
rate(node_network_receive_bytes_total[5m])
```

Host transmit:

```promql
rate(node_network_transmit_bytes_total[5m])
```

Container receive:

```promql
sum by(name)(
  rate(container_network_receive_bytes_total[5m])
)
```
