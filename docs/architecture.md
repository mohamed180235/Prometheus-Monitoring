# Architecture

The course uses a simple two-instance architecture.

```text
                    ┌──────────────────────┐
                    │       Grafana        │
                    │        :3000        │
                    └──────────┬───────────┘
                               │ PromQL
                               ▼
                    ┌──────────────────────┐
                    │     Prometheus       │
                    │        :9090        │
                    └──────────┬───────────┘
                               │ scrape
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
        Node Exporter                    cAdvisor
            :9100                          :8080
                 │                           │
                 └─────────────┬─────────────┘
                               ▼
                        EC2 #2 / Docker

Alert path:

Prometheus → Alertmanager → Notification
