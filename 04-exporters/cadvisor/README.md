# cAdvisor

cAdvisor is installed on EC2 #2 using Docker.

## Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

## Run cAdvisor

```bash
sudo docker run -d \
  --name=cadvisor \
  --restart=unless-stopped \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker:/var/lib/docker:ro \
  gcr.io/cadvisor/cadvisor:latest
```

Verify:

```bash
docker ps
curl localhost:8080/metrics
```

## PromQL

Container CPU:

```promql
rate(container_cpu_usage_seconds_total[5m])
```

Container memory:

```promql
container_memory_usage_bytes
```

Top 5 containers by CPU:

```promql
topk(
  5,
  sum by(name) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```
