# Step-by-step Prometheus Alertmanager Setup Guide 


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
vim alert.rules.yml
```
past configuration

```bash
groups:
- name: system_alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 80% for more than 5 minutes."

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High Memory usage on {{ $labels.instance }}"
      description: "Memory usage is above 80% for more than 5 minutes."

  - alert: HighDiskUsage
    expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High Disk usage on {{ $labels.instance }}"
      description: "Disk usage is above 80% for more than 5 minutes."

```

## Step 6: Prometheus configuration (prometheus.yml file)

```bash
global:
  scrape_interval: 10s

rule_files:
  - "/etc/prometheus/alert.rules.yml"

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
      - targets:
          - '10.10.8.159:9100'
          - '10.10.8.211:9100'
          - '10.10.8.176:9100'
          - '10.10.8.97:9100'
          - '10.10.8.48:9100'
          - '10.10.8.172:9100'
          - '10.10.8.114:9100'
          - '10.10.8.233:9100'
          - '10.10.8.224:9100'
          - '10.10.8.238:9100'
          - '10.10.0.179:9100'
          - '10.10.8.120:9100'
          - '10.10.0.250:9100'
          # Others Server ip 
```

Prometheus Alertmanager Restart

```bash
sudo systemctl restart prometheus
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
