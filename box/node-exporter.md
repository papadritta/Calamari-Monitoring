## 2. Node-exporter [Check the lastest release](https://github.com/prometheus/node_exporter/releases)

#### Installation

- create user
```
sudo useradd --no-create-home --shell /bin/false node_exporter
```

- download last version
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
```

- check sha256sum
```
sha256sum node_exporter-1.4.0.linux-amd64.tar.gz
```

- extract the archive
```
tar xvf node_exporter-1.4.0.linux-amd64.tar.gz
```

- copy the binary to the following location and set ownership
```
sudo cp node_exporter-1.4.0.linux-amd64/node_exporter /usr/local/bin
sudo chown -R node_exporter:node_exporter /usr/local/bin/node_exporter
```
- remove the download leftovers
```
rm -rf node_exporter-1.4.0.linux-amd64
rm node_exporter-1.4.0.linux-amd64.tar.gz
```
#### Setting service

- Create a systemd service
```
sudo nano /etc/systemd/system/node_exporter.service
```

- Copy & paste
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=node_exporter
Group=node_exporter
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/node_exporter --web.listen-address="localhost:9100"

[Install]
WantedBy=multi-user.target
```
- Reload systemd
```
sudo systemctl daemon-reload
```

- Start the service and check the status
```
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

- Enable Node Exporter to start on boot
```
sudo systemctl enable node_exporter
```
