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
