# Blackbox Exporter

Blackbox Exporter is used for probe-based monitoring such as:

- HTTP
- HTTPS
- TCP
- ICMP

The typical architecture is:

```text
Prometheus → Blackbox Exporter → Target
```

For this course, install Blackbox Exporter as a service and configure an HTTP module.

Example Prometheus scrape configuration:

```yaml
- job_name: blackbox-http
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
        - https://example.com
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: <BLACKBOX_HOST>:9115
```

Keep real production targets and credentials out of the repository.
