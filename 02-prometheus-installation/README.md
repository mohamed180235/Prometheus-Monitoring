# Module 02 — Installing Prometheus as a Service

Prometheus is installed on EC2 #1 and managed by systemd.

## 1. Create User and Directories

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

## 2. Install the Prometheus Binary

Download a current Prometheus Linux AMD64 release from the official Prometheus release page https://prometheus.io/download/, extract it, and install:
Download Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
tar -xvf prometheus-linux-amd64.tar.gz
cd prometheus-*/
```
Install Prometheus Binary
```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

Verify:

```bash
prometheus --version
promtool --version
```

## 3. Install Configuration

Copy:

```bash
sudo cp prometheus.yml /etc/prometheus/prometheus.yml
```

Set ownership:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

## 4. Install systemd Unit

```bash
sudo cp prometheus.service /etc/systemd/system/prometheus.service
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```

Check:

```bash
sudo systemctl status prometheus
```

## 5. Verify

Open:

```text
http://<PROMETHEUS_PUBLIC_IP>:9090
```

Go to:

**Status → Targets**

Run:

```promql
up
```
