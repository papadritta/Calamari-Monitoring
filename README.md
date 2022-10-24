# Monitoring &amp; Alerting solution for Calamari Network (Monitoring Stack)
![Copy of Staking is live](https://user-images.githubusercontent.com/90826754/197347767-66449f15-0dce-4538-ac82-2b8d626da6d9.jpg)

>**Calamari Network is a private layer built for the entire Kusama ecosystem. Built on the substrate framework, Calamari Network is natively compatible with other projects and parachain assets including wrapped major cryptoassets.**
>**Calamari, Manta Network's canary net, is the plug-and-play privacy-preservation parachain built to serve the Kusama world.**
>**The Calamari parachain will be Privacy Hub for Kusama with multiple privacy solutions for all web3 projects.**

>**Many individuals joined the adventure and set up their own validator node, which is extraordinary for decentralization. However, maintaining a validator node is a huge responsibility as you are basically securing millions of dollars on the blockchain. Health and security of your node have to be the top priority for you as a validator.**


**NOTE: THIS GUIDE PROVIDES HELPFUL MONITORING AND ALERTING SOLUTION FOR CALAMARI VALIDATORS, BUT MOST OF THE CONFIGURATION IS EXACTLY THE SAME FOR A POLKADOT, KUSAMA, OR ANY SUBSTRATE BASED NODE.**

<img width="2337" alt="Screen Shot 2022-10-23 at 22 48 37" src="https://user-images.githubusercontent.com/90826754/197495694-ccc10e8f-d8f4-49d1-8586-f1abce7cacae.png">

**Tested on Ubuntu 20.04.05 LTS**

## The set of services required:

## 1. Install and setup the UFW (rules for the ports) [Itstallation & setup](/ufw.md)

## 2. Node-exporter [Itstallation & setup](/node-exporter.md)

## 3. Prometheus [Itstallation & setup](/prometheus.md)

## 4. Grafana [Itstallation & setup](/grafana.md)

## 5. Page duty intagration [Itstallation & setup](/pageduty.md)

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

**NOTE: Don't forget to change <add_your_key_here> from Page Duty and add your key.**

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

<img width="1857" alt="Screen Shot 2022-10-23 at 13 54 48" src="https://user-images.githubusercontent.com/90826754/197448580-c260097b-89a6-4013-9df7-ed7a729c4c72.png">

1. Download JSON file
Standart Node-Exporter Dashboard you can download [here](https://www.acv.is/)

2. Upload JSON file or Import via grafana.com by ID, type 1860 and click on the Load button.
<img width="1583" alt="Screen Shot 2022-10-24 at 11 33 51" src="https://user-images.githubusercontent.com/90826754/197449464-8c457796-3c89-4547-8675-a330135853a8.png">

3. Run Node-Exporter Dashboard & setup the dashboard as you wish
<img width="1812" alt="Screen Shot 2022-10-24 at 16 11 36" src="https://user-images.githubusercontent.com/90826754/197492576-4ef3f3d5-91e9-4676-8210-f45b8cb041cc.png">

ALL DONE!!!

**NOTE: IF YOU NEED PROFESSIONAL CUSTOM DASHBOARD, PLEASE REACH ME OUT**

<img width="2337" alt="Screen Shot 2022-10-23 at 22 48 37" src="https://user-images.githubusercontent.com/90826754/197448743-b2598483-79c5-48d1-bb30-a92180c8915d.png">

