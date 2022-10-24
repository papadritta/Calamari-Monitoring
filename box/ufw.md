
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
