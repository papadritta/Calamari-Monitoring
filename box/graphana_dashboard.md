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
