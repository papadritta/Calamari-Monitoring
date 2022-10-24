## 3. Prometheus [Check the lastest release](https://github.com/prometheus/prometheus/releases)

### Instalation

- create user
```
sudo useradd --no-create-home --shell /bin/false prometheus
```
- create directories & chown ownership
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
```
- download last version
```
wget https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz
```

- check sha256sum
```
sha256sum prometheus-2.39.1.linux-amd64.tar.gz
```

- extract the archive
```
tar xvf prometheus-2.39.1.linux-amd64.tar.gz
```

- copy the binary to the following location and set ownership
```
sudo cp prometheus-2.39.1.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.39.1.linux-amd64/promtool /usr/local/bin/
sudo chown -R prometheus:prometheus /usr/local/bin/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/promtool
```

- copy the content files to the following locations and set ownership
```
sudo cp -r prometheus-2.39.1.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.39.1.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.39.1.linux-amd64/console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

- remove the download leftovers
```
rm -rf prometheus-2.39.1.linux-amd64
rm prometheus-2.39.1.linux-amd64.tar.gz
```
### Setting config

- Setup YAML config
```
sudo nano /etc/prometheus/prometheus.yml
```
-copy and paste
```
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
      
rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'calamari'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9615']
```

- set ownership for the config file
```
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
```

- test the service
```
sudo -u prometheus /usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address="localhost:9090"
```

### Setting service

- Create a systemd service
```
sudo nano /etc/systemd/system/prometheus.service
```
- copy and paste
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address="localhost:9090"
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

- Reload systemd
```
sudo systemctl daemon-reload
```
- start the service and check the status
```
sudo systemctl start prometheus
sudo systemctl status prometheus
```

- enable prometheus to start on boot
```
sudo systemctl enable prometheus
```
