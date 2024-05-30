# Node Exporter

## Purpose

node_exporter is a Prometheus exporter specifically designed for monitoring the system metrics of *nix systems, such as Linux. It collects hardware and OS-level metrics and exposes them to Prometheus, enabling the monitoring of the host system

## Download

```bash
VERSION=1.8.1
wget https://github.com/prometheus/node_exporter/releases/download/v${VERSION}/node_exporter-${VERSION}.linux-amd64.tar.gz
tar xvf node_exporter-${VERSION}.linux-amd64.tar.gz
```

## Use systemd

It's basically the same procedure as prometheus

* Copy the binary: `sudo cp node_exporter /usr/local/bin/`
* Create user: `useradd --no-create-home --shell /bin/false node_exporter`
* Set permission: `sudo chown node_exporter. /usr/local/bin/node_exporter`
* Set the system configuration file:
  ```bash
  echo "[Unit]
  Description=Node Exporter
  Wants=network-online.target
  After=network-online.target

  [Service]
  User=node_exporter
  Group=node_exporter
  Type=simple
  ExecStart=/usr/local/bin/node_exporter

  [Install]
  WantedBy=multi-user.target
  " | sudo tee /etc/systemd/system/node_exporter.service
  ```
