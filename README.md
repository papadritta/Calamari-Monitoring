# Monitoring &amp; Alerting solution for Calamari Network (Monitoring Stack)
![Copy of Staking is live](https://user-images.githubusercontent.com/90826754/197347767-66449f15-0dce-4538-ac82-2b8d626da6d9.jpg)

>**The Polkadot & Kusama ecosystems have raised an incredible attention in the past few years.**

>**Calamari Network is a private layer built for the entire Kusama ecosystem. Built on the substrate framework, Calamari Network is natively compatible with other projects and parachain assets including wrapped major cryptoassets.**

>**Calamari, Manta Network's canary net, is the plug-and-play privacy-preservation parachain built to serve the Kusama world.**

>**The Calamari parachain will be Privacy Hub for Kusama with multiple privacy solutions for all web3 projects.**

>**Many individuals joined the adventure and set up their own validator node, which is extraordinary for decentralization. However, maintaining a validator node is a huge responsibility as you are basically securing millions of dollars on the blockchain. Health and security of your node have to be the top priority for you as a validator.**



**NOTE: THIS GUIDE PROVIDES HELPFUL MONITORING AND ALERTING SOLUTION FOR CALAMARI VALIDATORS, BUT MOST OF THE CONFIGURATION IS EXACTLY THE SAME FOR A POLKADOT, KUSAMA, OR ANY SUBSTRATE BASED NODE.**

**Tested on Ubuntu 20.04**

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

## 4. Grafana

#### Installation

- Install the prerequisites
```
sudo apt update -y
sudo apt install -y apt-transport-https software-properties-common wget
```

- Import the Grafana PGP key
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo tee /etc/apt/trusted.gpg.d/grafana.asc
```

- Add the Grafana OSS APT repository for stable releases
```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

- Install Grafana from the APT repository
```
sudo apt update
sudo apt install grafana
```

- Open the Grafana configuration file
```
sudo nano /etc/grafana/grafana.ini
```

> Search for the following section
```
[server]
# Protocol (http, https, h2, socket)
;protocol = http

# The ip address to bind to, empty will bind to all interfaces
;http_addr =
```

> Modify the configuration file to
```   
[server]
# Protocol (http, https, h2, socket)
;protocol = http

# The ip address to bind to, empty will bind to all interfaces
http_addr = localhost
```

- Reload systemd
```
sudo systemctl daemon-reload
```

- start the service and check the status
```
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

- enable Grafana to start on boot
```
sudo systemctl enable grafana-server
```
## 5. Page duty intagration [Page Duty](https://www.pagerduty.com/)

<img width="1623" alt="1" src="https://user-images.githubusercontent.com/90826754/197352204-883fd91e-59fb-47a6-80b8-f6e89a4bf749.png">

- signup with an email

- choose your region

- set a service name

- skip intagration on first step

- add your phone number for sms & areting call and test it

- choose your service and add intagration Event API V2
<img width="1620" alt="6" src="https://user-images.githubusercontent.com/90826754/197352258-d66e9597-4f2c-4ff3-9595-aee70dff22fd.png">

- open the tab and copy Integration Key (<add_your_key_here>)

**NOTE: this key we need to add to Setup the Alertmanager YAML configuration file bellow**

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
- copy and paste (don't forget to change <add_your_key_here> from Page Duty and add your key)
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

## 7. Setting Alert Rules for Prometheus

### Setting config

- Setup Alert Rules configuration file for Prometheus
``` 
sudo nano /etc/prometheus/alert_rules.yml
```
- Copy and paste
```
groups:
- name: alert_rules
  rules:
  - alert: Available_disk_space_too_low
    expr: node_filesystem_avail_bytes{mountpoint="/"} <= 82830000000
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Available disk space below 80GB
  - alert: Available_memory_too_low
    expr: node_memory_MemAvailable_bytes <= 1024000000
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Available memory below 1GB
  - alert: CPU_usage_too_high
    expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) >= 90
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: CPU usage above 90%
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: Calamari node is down more than 1 minute!!!
```
- check the rules with promtool
```
promtool check rules /etc/prometheus/alert_rules.yml
```
- Restart prometheus and check status
```
sudo systemctl restart prometheus.service
sudo systemctl status prometheus.service
```
## 8. Setting Graphana dashboard

### Accessing Grafana and adding your dashboards:

- check you server IP <server_IP> by running command
```
wget -qO- eth0.me
```

**NOTE: The <server_IP> output add to the next line**


- login on your browser
```
http://<server_IP>:3000/
```

**NOTE: The default credentials for Grafana are: Username: admin, Password: admin**

<img width="2036" alt="Screen Shot 2022-10-23 at 12 54 51" src="https://user-images.githubusercontent.com/90826754/197377204-250c3a81-58ac-4e94-958a-aecce023e756.png">

- make a new strong password

- login into your Graphana dashboard with your new credenteals

<img width="2026" alt="Screen Shot 2022-10-23 at 12 57 17" src="https://user-images.githubusercontent.com/90826754/197377286-1142beba-8059-46ed-9bac-317c3ff4aed0.png">

- Add your first data source (Prometheus)
```
> Data Sources / Prometheus
add URL http://localhost:9090
```
<img width="1736" alt="Screen Shot 2022-10-23 at 13 02 43" src="https://user-images.githubusercontent.com/90826754/197377373-cd604752-04dd-42ea-8010-4b5b5e1ade6f.png">

- Save & Test 
> Data source is working

<img width="1850" alt="Screen Shot 2022-10-23 at 13 03 10" src="https://user-images.githubusercontent.com/90826754/197377392-6382954d-f82d-45b7-9f3a-dc3b511d52a1.png">

- Add your second data source (Alertmanager)
```
> Data Sources / Alertmanager
add Implementation Prometheus
add URL http://localhost:9093
```
<img width="1747" alt="Screen Shot 2022-10-23 at 13 11 23" src="https://user-images.githubusercontent.com/90826754/197377459-6b7173d4-33fc-4200-8e32-c299b323f03d.png">

- Save & Test 
> Data source is working

<img width="1770" alt="Screen Shot 2022-10-23 at 13 10 54" src="https://user-images.githubusercontent.com/90826754/197377477-98843074-d5fe-4df6-b1f6-b0521a0b7a63.png">

- Add your first Dashboard





