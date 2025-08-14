# Step-by-step Prometheus Alertmanager Setup Guide 

## Prerequisites List
1. Node Exporter installed on all target servers
2. Prometheus & Grafana installed on main monitoring server
3. Gmail App Password generated
    - Gmail 2FA enabled
    - App password Alertmanager config
4. Open Firewall Ports
    - 9090 (Prometheus)
    - 9093 (Alertmanager)
    - 9100 (Node Exporter)
    - Grafana port (default 3000)

![](https://github.com/abrahimcse/prometheus-alertmanager-setup-guide/blob/main/alert.jpeg)

## Step 1: Alertmanager Download and Install

```bash
cd /opt
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
tar xvf alertmanager-0.27.0.linux-amd64.tar.gz
mv alertmanager-0.27.0.linux-amd64 alertmanager
```

## Step 2: Alertmanager Configuration 
```bash
vim /opt/alertmanager/alertmanager.yml
```

Run Alertmanager

```bash
cd /opt/alertmanager
./alertmanager --config.file=alertmanager.yml &
```

## Step 5: Alert Rules create (alert.rules.yml)

```bash
cd /ctech/prometheus
vim alert.rules.yml
```

## Step 6: Prometheus configuration (prometheus.yml file)

```bash
cd /ctech/prometheus
vim prometheus.yml
```

**Prometheus Alertmanager Restart**

```bash
cd /ctech/prometheus/
docker compose down
docker compose up -d
```

## Step 7: Systemd create for alert message

```bash
vim /etc/systemd/system/alertmanager.service
```
past

```bash
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/opt/alertmanager/alertmanager \
  --config.file=/opt/alertmanager/alertmanager.yml \
  --storage.path=/opt/alertmanager/data

[Install]
WantedBy=multi-user.target

```

```bash
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
```

## Step 8: Test & Verify
- Browser : http://your-server-ip:9093
- Prometheuse Alert Rules from UI : http://your-server-ip:9090/alerts

Give a cpu load
```bash
sudo apt install stress -y
stress --cpu 4 --timeout 300
```
