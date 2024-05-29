
# Installation
## Download

* Go to the [download page](https://prometheus.io/download/)

* Download and install
    ```bash
    # Download the version you want
    VERSION=2.52.0
    wget https://github.com/prometheus/prometheus/releases/download/v${VERSION}/prometheus-${VERSION}.linux-amd64.tar.gz

    # Untar the file
    tar xvf prometheus-${VERSION}.linux-amd64.tar.gz
    ```

* Within the extracted file you find:
    ```bash
    ls -Al prometheus-2.52.0.linux-amd64

    total 262508
    drwxr-xr-x 2 mike mike      4096 mai    9 00:11 console_libraries
    drwxr-xr-x 2 mike mike      4096 mai    9 00:11 consoles
    -rw-r--r-- 1 mike mike     11357 mai    9 00:11 LICENSE
    -rw-r--r-- 1 mike mike      3773 mai    9 00:11 NOTICE
    -rwxr-xr-x 1 mike mike 138438117 mai    8 23:58 prometheus
    -rw-r--r-- 1 mike mike       934 mai    9 00:11 prometheus.yml
    -rwxr-xr-x 1 mike mike 130329948 mai    8 23:58 promtool
    ```
    * `prometheus`: executable
    * `prometheus.yml`: configuration file
    * `promtool`: command-line utility

## Running a prometheus instance

* Launch the vanilla prometheus: `./prometheus`
* Open a web browser at http://localhost:9090

## Installation with systemd
### Configure prometheus requirement

Instead of running prometheus through the binary, let's use systemd to manage it:
* Create a dedicated user: `sudo useradd --no-create-home --shell /bin/false prometheus`
  * `--no-create-home`: no home directory
  * `--shell /bin/false`: user can't login
* Create a configuration directory: `sudo mkdir /etc/prometheus`
* Create a directory for the TSDB: `sudo mkdir /var/lib/prometheus`
* Copy the prometheus executable and the promtool you downloaded to `sudo cp prometheus promtool /usr/local/bin/`
* Set the permission: `sudo chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}`
* Copy the consoles and console_libraries folders: `sudo cp -r consoles console_libraries /etc/prometheus`
* Copy the config file: `sudo cp prometheus.yml /etc/prometheus/prometheus.yml`
* Set the permission: `sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus`

To run prometheus with this settings:
```bash
sudo -u prometheus /usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console
  --web.console.libraries=/etc/prometheus/console_libraries
```

### Configure prometheus in systemd

```
echo "[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
" | sudo tee /etc/systemd/system/prometheus.service
```

Run prometheus using systemd:
```bash
sudo systemctl daemon-reload 
sudo systemctl start prometheus.service
sudo systemctl status prometheus.service
```
The service is now running
```bash
● prometheus.service - Prometheus
     Loaded: loaded (/etc/systemd/system/prometheus.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-05-29 07:53:31 CEST; 1s ago
   Main PID: 30532 (prometheus)
      Tasks: 9 (limit: 19017)
     Memory: 17.2M
        CPU: 46ms
     CGroup: /system.slice/prometheus.service
     ...
```