# Troubleshooting

## Prometheus target is DOWN

Check:

```bash
curl http://<TARGET_PRIVATE_IP>:9100/metrics
curl http://<TARGET_PRIVATE_IP>:8080/metrics
```

Check services:

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo docker ps
```

Check security groups and private IP addresses.

## Validate Prometheus config

```bash
promtool check config /etc/prometheus/prometheus.yml
```

## Validate alert rules

```bash
promtool check rules /etc/prometheus/rules/*.yml
```

## Prometheus logs

```bash
sudo journalctl -u prometheus -f
```

## Node Exporter logs

```bash
sudo journalctl -u node_exporter -f
```

## cAdvisor logs

```bash
docker logs cadvisor
```
