## 6. Alert-Manager [Check the lastest release](https://github.com/prometheus/alertmanager/releases)

### Instalation
- Create the alertmanager user
```
sudo useradd --no-create-home --shell /bin/false alertmanager
```

- Create the configuration and data directories with proper ownership
```
sudo mkdir /etc/alertmanager
sudo mkdir /var/lib/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown -R alertmanager:alertmanager /var/lib/alertmanager
```

- Download the latest stable version of Alertmanager
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
```

- check SHA256 Checksum
```
sha256sum alertmanager-0.24.0.linux-amd64.tar.gz
```

- Extract the archive
```
tar xvf alertmanager-0.24.0.linux-amd64.tar.gz
```

- Copy the binaries to the following locations and set ownership
```
sudo cp alertmanager-0.24.0.linux-amd64/alertmanager /usr/local/bin/
sudo cp alertmanager-0.24.0.linux-amd64/amtool /usr/local/bin/
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```

- Remove the download leftovers
```
rm -rf alertmanager-0.24.0.linux-amd64
rm alertmanager-0.24.0.linux-amd64.tar.gz
```
### Setting config

- Setup the Alertmanager YAML configuration file 
```
sudo nano /etc/alertmanager/alertmanager.yml
```
- copy and paste 

**NOTE: Don't forget to change [<add_your_key_here>](/box/pageduty.md) from Page Duty and add your key.** 

```
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'pagerduty'

receivers:
- name: 'pagerduty'
  pagerduty_configs:
  - routing_key: <add_your_key_here>

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

- Set ownership for the config file
```
sudo chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```
### Setting service

- Setup the Alertmanager systemd service. Open the service definition file
```
sudo nano /etc/systemd/system/alertmanager.service
```
- Copy and paste
```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=alertmanager
Group=alertmanager
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/alertmanager \
    --config.file /etc/alertmanager/alertmanager.yml \
    --storage.path /var/lib/alertmanager/ \
    --web.listen-address="localhost:9093" \
    --cluster.listen-address=""
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

- Reload systemd to reflect the changes
```
sudo systemctl daemon-reload
```

-Start the service and check the status
```
sudo systemctl start alertmanager.service
sudo systemctl status alertmanager.service
```

- Enable the Alertmanager service to start on boot
```
sudo systemctl enable alertmanager.service
```
