# Module 07 — Alerting

## Alerting Architecture

```text
Prometheus
   │
   │ evaluates rules
   ▼
Alert Rule
   │
   ▼
PENDING → FIRING
   │
   ▼
Alertmanager
   │
   ├── Group
   ├── Route
   ├── Silence
   └── Inhibit
```

## Prometheus Alert States

```text
INACTIVE → PENDING → FIRING
```

The `for` field controls how long the expression must remain true before the alert fires.

## Install Alert Rules

```bash
sudo mkdir -p /etc/prometheus/rules
sudo cp prometheus/alert-rules.yml /etc/prometheus/rules/
```

Validate:

```bash
promtool check rules /etc/prometheus/rules/alert-rules.yml
```

## Configure Alertmanager

Install Alertmanager as a systemd service and configure Prometheus:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093
```

## Generate CPU Load

```bash
docker run -d \
  --name cpu-stress \
  ubuntu \
  bash -c "while true; do :; done"
```

Remove:

```bash
docker rm -f cpu-stress
```

## Important

Notification receiver examples in this repository intentionally avoid real webhook URLs or credentials.
