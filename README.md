# Prometheus Monitoring Course

Hands-on Prometheus, PromQL, Grafana, Exporters, and Alertmanager course labs.

## Lab Architecture

```text
                         AWS

              EC2 #1 — Monitoring Server
        ┌─────────────────────────────────┐
        │ Prometheus :9090                │
        │ Grafana    :3000                │
        │ Alertmanager :9093              │
        └───────────────┬─────────────────┘
                        │
                    scrape over
                  private network
                        │
                        ▼
              EC2 #2 — Target Server
        ┌─────────────────────────────────┐
        │ Node Exporter :9100             │
        │ cAdvisor      :8080             │
        │ Docker containers               │
        └─────────────────────────────────┘
```

## Course Modules

| Module | Topic |
|---|---|
| 01 | Monitoring Fundamentals |
| 02 | Prometheus Installation |
| 03 | Configuration & Service Discovery |
| 04 | Exporters |
| 05 | PromQL |
| 06 | Grafana |
| 07 | Alerting |

## Recommended Order

1. Read `docs/aws-setup.md`
2. Complete Module 01
3. Install Prometheus as a systemd service in Module 02
4. Configure targets and service discovery
5. Install Node Exporter and cAdvisor
6. Practice PromQL
7. Build the Grafana dashboard
8. Configure Prometheus alerts and Alertmanager
9. Complete the labs

## Important

These demos are designed for AWS EC2. Replace placeholders such as `<PROMETHEUS_PRIVATE_IP>` with your own values.

**Never commit AWS private keys, passwords, tokens, webhook URLs, or real secrets.**
