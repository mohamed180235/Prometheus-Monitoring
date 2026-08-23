# Module 03 — Configuration & Service Discovery

## Static Targets

The basic configuration uses:

```yaml
scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - <TARGET_PRIVATE_IP>:9100
```

## Multiple Targets

```yaml
  - job_name: node
    static_configs:
      - targets:
          - <TARGET_PRIVATE_IP_1>:9100
          - <TARGET_PRIVATE_IP_2>:9100
```

## File-Based Discovery

Create a file such as:

```text
targets/nodes.json
```

Example:

```json
[
  {
    "targets": [
      "<TARGET_PRIVATE_IP>:9100"
    ],
    "labels": {
      "environment": "lab",
      "team": "students"
    }
  }
]
```

Then:

```yaml
- job_name: node-file-discovery
  file_sd_configs:
    - files:
        - /etc/prometheus/targets/*.json
```

## Reload Configuration

Validate:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

For a Prometheus process configured with the lifecycle endpoint, a configuration reload can be requested without restarting the service.

In a classroom lab, restarting systemd is also acceptable:

```bash
sudo systemctl restart prometheus
```

## Exercise

Add a second target and verify both targets are `UP`.
