# Monitoring &amp; Alerting solution for Calamari Network (Monitoring Stack)

>**The Polkadot & Kusama ecosystems have raised an incredible attention in the past few years.**

>**Calamari Network is a private layer built for the entire Kusama ecosystem. Built on the substrate framework, Calamari Network is natively compatible with other projects and parachain assets including wrapped major cryptoassets.**

>**Calamari, Manta Network's canary net, is the plug-and-play privacy-preservation parachain built to serve the Kusama world.**

>**The Calamari parachain will be Privacy Hub for Kusama with multiple privacy solutions for all web3 projects.**

>**Many individuals joined the adventure and set up their own validator node, which is extraordinary for decentralization. However, maintaining a validator node is a huge responsibility as you are basically securing millions of dollars on the blockchain. Health and security of your node have to be the top priority for you as a validator.**



**NOTE: THIS GUIDE PROVIDES HELPFUL MONITORING AND ALERTING SOLUTION FOR CALAMARI VALIDATORS, BUT MOST OF THE CONFIGURATION IS EXACTLY THE SAME FOR A POLKADOT, KUSAMA, OR ANY SUBSTRATE BASED NODE.**



## The set of services required:

## 1. Install and setup the UFW (rules for the ports)

- Install UFW
```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y build-essential && sudo apt install jq -y
apt-get install lsof ufw
```
- Setup and enable the rules

```
ufw allow 22
ufw allow 80
ufw allow 443
ufw allow 3000
ufw allow 9090
ufw allow 9093
ufw allow 9100
ufw allow 9133
ufw allow 9144
ufw allow 9615
ufw allow 9616
ufw allow 9933
ufw allow 30333
ufw allow 30334
ufw allow 31333
ufw allow 31334

ufw deny out from any to 10.0.0.0/8
ufw deny out from any to 172.16.0.0/12
ufw deny out from any to 192.168.0.0/16
ufw deny out from any to 100.64.0.0/10
ufw deny out from any to 198.18.0.0/15
ufw deny out from any to 169.254.0.0/16
```
**NOTE: This is general rules for UFW. Don't forget to set additional rules for you SSH-client (if you use port tunneling) and NGINX (if you use proxi SSL).**

- enable UFW and check the status
```
ufw enable
```

- check UFW, listen specific and all ports (use command one by one)
```
sudo ufw status
sudo lsof -i -P -n | grep LISTEN | grep calamari
ss -tulpn
```

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

-copy the binary to the following location and set ownership
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



