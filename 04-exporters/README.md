# Module 04 — Exporters

## Objectives

- Understand exporters
- Install Node Exporter
- Install cAdvisor
- Understand `/metrics`
- Collect CPU, memory, disk, and network metrics
- Understand host metrics vs container metrics

## Node Exporter

Node Exporter exposes host-level metrics on port `9100`.

Test:

```bash
curl http://localhost:9100/metrics
```

Useful metrics:

```promql
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_memory_MemTotal_bytes
node_filesystem_avail_bytes
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

## cAdvisor

cAdvisor exposes container metrics on port `8080`.

Test:

```bash
curl http://localhost:8080/metrics
```

Useful metrics:

```promql
container_cpu_usage_seconds_total
container_memory_usage_bytes
container_network_receive_bytes_total
container_network_transmit_bytes_total
```

## Demo: Generate a CPU-Heavy Container

```bash
docker run -d \
  --name cpu-stress \
  ubuntu \
  bash -c "while true; do :; done"
```

Remove it:

```bash
docker rm -f cpu-stress
```
