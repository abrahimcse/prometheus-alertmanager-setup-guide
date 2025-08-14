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

Past this code

```bash
global:
  resolve_timeout: 5m

route:
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: 'abrahimcse@gmail.com, moyeencse@gmail.com, gokam@gmail.com'
        from: 'abrahim.ctech@gmail.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'abrahim.ctech@gmail.com'
        auth_identity: 'abrahim.ctech@gmail.com'
        auth_password: 'your app pass'
        require_tls: true

```
Run Alertmanager

```bash
cd /opt/alertmanager
./alertmanager --config.file=alertmanager.yml &
````

## Step 5: Alert Rules create (alert.rules.yml)

```bash
cd /ctech/prometheus
vim alert.rules.yml
```
past configuration

```bash
groups:
  - name: instance-alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}({{ $labels.instance }})"
          description: "CPU usage is above 80% for more than 2 minutes on {{ $labels.name }}({{ $labels.instance }})"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 80% for more than 2 minutes on {{ $labels.name }}({{ $labels.instance }})"

      - alert: HighDiskUsage
        expr: (1 - (node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes{fstype!~"tmpfs|overlay"})) * 100 > 80
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High disk usage on {{ $labels.instance }}"
          description: "Disk usage is above 80% for more than 2 minutes on {{ $labels.name }}({{ $labels.instance }})"

```

## Step 6: Prometheus configuration (prometheus.yml file)

```bash
cd /ctech/prometheus
vim prometheus.yml
```

```bash
global:
  scrape_interval: 10s

rule_files:
  - "alert.rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['10.10.8.159:9100']
        labels:
          name: 'stg-k8s-master'
      - targets: ['10.10.8.211:9100']
        labels:
          name: 'stg-k8s-worker-1'
      - targets: ['10.10.8.176:9100']
        labels:
          name: 'stg-kafka'
          # Others Server ip 
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
