# AWS Lab Setup

## EC2 Instances

Create two Ubuntu EC2 instances.

### EC2 #1 — Monitoring Server

Install:

- Prometheus
- Grafana
- Alertmanager

Recommended size for a small class lab: `t3.small` or similar.

### EC2 #2 — Target Server

Install:

- Node Exporter
- Docker
- cAdvisor

Recommended size: `t3.micro` or similar.

## Security Groups

### Monitoring Server

Allow:

| Port | Source | Purpose |
|---|---|---|
| 22 | Your IP | SSH |
| 9090 | Your IP | Prometheus UI |
| 3000 | Your IP | Grafana |
| 9093 | Your IP | Alertmanager UI |

### Target Server

Allow:

| Port | Source | Purpose |
|---|---|---|
| 22 | Your IP | SSH |
| 9100 | Monitoring Server SG | Node Exporter |
| 8080 | Monitoring Server SG | cAdvisor |

Do not expose exporter ports to `0.0.0.0/0`.

## Connectivity Test

From EC2 #1:

```bash
curl http://<TARGET_PRIVATE_IP>:9100/metrics
curl http://<TARGET_PRIVATE_IP>:8080/metrics
```

Both should return Prometheus-format metrics.
