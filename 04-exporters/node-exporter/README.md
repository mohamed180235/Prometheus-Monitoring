# Node Exporter

Install on EC2 #2.

Create the user:

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

Install the current `node_exporter` binary from the official release, then:

```bash
sudo cp node_exporter /usr/local/bin/
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Install the included systemd unit:

```bash
sudo cp node_exporter.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter
```

Verify:

```bash
sudo systemctl status node_exporter
curl localhost:9100/metrics
```
