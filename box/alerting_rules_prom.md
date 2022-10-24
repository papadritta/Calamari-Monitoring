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
