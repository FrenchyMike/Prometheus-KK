# Autentication and encryption

## Encryption

* Create a certificate
  ```
  sudo openssl req -new \
    -newkey rsa:2048 \
    -days 365 \
    -nodes -x509 \
    -keyout node_exporter.key -out node_exporter.crt \
    -subj "/C=FR/L=Nice/O=MyOrg/CN=localhost" \
    -addext "subjectAltName = DNS:localhost"
  ```
  It provides a pair a file: `node_exporter.crt` and `node_exporter.crt`

* Create a config file: `vim config.yml`
  ```yaml
  tls_server_config:
    cert_file: node_exporter.crt
    key_file: node_exporter.key
  ```
* Create a configuration directory:
  ```bash
  sudo mkdir /etc/node_exporter
  sudo cp config.yml node_exporter.* /etc/node_exporter/
  sudo chown -R node_exporter. /etc/node_exporter
  ```
* Update node_exporter system file: `sudo vim /etc/systemd/system/node_exporter.service`, at the `ExecStart` line:
  ```
  [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target

  [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    ExecStart=/usr/local/bin/node_exporter --web.config.file=/etc/node_exporter/config.yml

  [Install]
    WantedBy=multi-user.target
  ```

* Restart the service and check the status
  ```bash
  sudo systemctl daemon-reload
  sudo systemctl restart node_exporter.service
  sudo systemctl status node_exporter.service 
  ```

* Check the URL: `curl -k https://localhost:9100/metrics`
  * `-k` make curl allows self signed certificates

* Update prometheus configuration
  * Copy the certificate to the prometheus config file: `sudo -u prometheus cp node_exporter.crt /etc/prometheus/`
  * Add `https` to the `scheme` an `tls_config` to the config file:
    ```yaml
    scrape_configs:
    - job_name: "nodes" # A job consists of a collection of instances that need to be scraped
      # Configs for scrape job. Takes a precedence over global config
      scrape_interval: 30s
      scrape_timeout: 3s
      scheme: https
      tls_config:
        ca_file: /etc/prometheus/node/exporter.crt
        insecure_skip_verify: true
      metrics_path: /metrics
      static_configs:
        - targets: ["localhost:9100"]
    ```

* Restart prometheus: `sudo systemctl restart prometheus`

## Authentication
* Generate a hash password
  * Using `apache2-utils`
    ```bash
    sudo apt install -y apache2-utils
    htpasswd -nBC 12 "" | tr -d ':\n'
    New password: 
    Re-type new password: 
    $2y$12$daXjcVE9mzRbgpkFVPhAFekLCaRVau2H0BvsTjpwiE4qkyVjWCVSu
    ```
* Update the node_exporter config file
  ```yaml
  tls_server_config:
    cert_file: node_exporter.crt
    key_file: node_exporter.key
  basic_auth_users:
    prometheus: $2y$12$daXjcVE9mzRbgpkFVPhAFekLCaRVau2H0BvsTjpwiE4qkyVjWCVS
  ```
* Restart the service: `sudo systemctl restart node_exporter.service`
* Add the user, password to the prometheus configuration file: `sudo vim /etc/prometheus/prometheus.yml`:
  ```yaml
    - job_name: "nodes"
      scrape_interval: 30s
      scrape_timeout: 3s
      scheme: https
      tls_config:
        ca_file: /etc/prometheus/node_exporter.crt
        insecure_skip_verify: true
      metrics_path: /metrics
      basic_auth:
        username: prometheus
        password: prometheus
  ```
* Restart the service: `sudo systemctl restart prometheus.service`
