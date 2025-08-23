:red_square: __Prometheus Installation__ \
:red_square: __Grafana Installation__ \
:red_square: __Node Exporter Installation__ \
:red_square: __Node Exporter Installation in client Machines__ \
:red_square: __Alert Manager Installation__ \
:red_square: __scrape config__ \
:red_square: __SMTP config for Alerts__ \
:red_square: __Alert Manager rules__ 



:blue_square: <ins>**This text is bold and underlined.**</ins>
```
apt update -y && apt upgrade -y
```

Download the latest prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz
```
```
tar -xvf prometheus-3.5.0.linux-amd64.tar.gz
```
rename the prometheus-3.5.0.linux-amd64 to prometheus-files
```
mv prometheus-3.5.0.linux-amd64 prometheus-files
```
Add a Prometheus user
```
useradd --no-create-home --shell /bin/false prometheus
```
create necessary directories
```
mkdir /etc/prometheus
mkdir /var/lib/prometheus
```
Change the owner of the above directories
```
chown prometheus:prometheus /etc/prometheus
```
```
chown prometheus:prometheus /var/lib/prometheus
```
Copy prometheus and promtool binary from prometheus-files folder to /usr/local/bin and change the ownership to prometheus user
```
cp prometheus-files/prometheus /usr/local/bin/
cp prometheus-files/promtool /usr/local/bin/
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
```
Setup Prometheus Configuration
Create the prometheus.yml file.
```
vim /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "/etc/prometheus/rules/*.yml"


scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'prometheus_server'
    scrape_interval: 5s
    static_configs:
      - targets: ['prometheus_server_ip:9100']

  - job_name: 'node-1'
    scrape_interval: 5s
    static_configs:
      - targets: ['node-1_server_ip:9100']
```
Change the ownership of the file to prometheus user
```
chown prometheus:prometheus /etc/prometheus/prometheus.yml
```
Setup Prometheus Service File
Create a prometheus service file.
```
vim /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
       --config.file /etc/prometheus/prometheus.yml \
       --storage.tsdb.path /var/lib/prometheus/ \
       --storage.tsdb.retention.time=7d \
       --storage.tsdb.retention.size=8GB \
       --web.console.templates=/etc/prometheus/consoles \
       --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
```
systemctl daemon-reload && systemctl start prometheus && systemctl enable prometheus && systemctl status prometheus --no-pager
```
access the prometheus UI on 9090 port \
http://prometheus_server_ip:9090/graph

:blue_square: __Grafana Installation__
update the server
```
apt update -y
```
install dependencies
```
sudo apt install -y adduser libfontconfig1 musl
```
download the grafana package
```
wget https://dl.grafana.com/oss/release/grafana_12.1.0_amd64.deb
```
```
ls
```
install the package
```
sudo dpkg -i grafana_12.1.0_amd64.deb
```
reload the daemon, enable, start and check status
```
sudo /bin/systemctl daemon-reload && sudo /bin/systemctl enable grafana-server && sudo /bin/systemctl start grafana-server && sudo /bin/systemctl status grafana-server
```
to login to grafana dashboard \
http://grafana_server_ip:3000/login


:blue_square: __Node Exporter Installation__
download the source
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
```
```
tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz
```
move the folder to the specified location
```
mv node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/
```
add the user
```
useradd -rs /bin/false node_exporter
```
create the node_exporter service file
```
vim /etc/systemd/system/node_exporter.service
```
```
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
```
reload the daemon , start , enable and check the status of node_exporter
```
systemctl daemon-reload && systemctl start node_exporter && systemctl enable node_exporter && systemctl status node_exporter --no-pager
```
in the browser \
http://server_IP:9100/metrics



:blue_square: __Node Exporter Installation in Client Machines__
install node_exporter in all client machines

:blue_square: __Add Data sources__
add data sources \
add node exporter full 1860 id for dashboard


:blue_square: __Prometheus Alert Manager Installation in Prometheus Server__

update the server
```
apt update -y
```
Download the alertmanager
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
```
```
tar -xvf alertmanager-0.28.1.linux-amd64.tar.gz
```
```
cd alertmanager-0.28.1.linux-amd64/
```
```
cp -r . /usr/local/bin/alertmanager
```
create the alertmanager service file
```
vim /etc/systemd/system/alertmanager.service
```
```
[Unit]
Description=Prometheus Alert Manager Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager \
    --config.file=/usr/local/bin/alertmanager/alertmanager.yml \
    --cluster.advertise-address="server_ip:9093"

[Install]
WantedBy=multi-user.target
```
check the alertmanager config with amtool
```
/usr/local/bin/alertmanager/amtool check-config /usr/local/bin/alertmanager/alertmanager.yml
```
reload the daemon , start , enable and check the status of alertmanager server
```
systemctl daemon-reload && systemctl start alertmanager.service && systemctl enable alertmanager.service && systemctl status alertmanager.service --no-pager
```
in browser \
http://server_ip:9093


:blue_square: __Notification Config for Alert Manager__
__for Email Notification__
```
vim /usr/local/bin/alertmanager/alertmanager.yml
```
```
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 24h
  receiver: 'email'
receivers:
- name: 'email'
  email_configs:
  - to: tkdhanasekar@gmail.com,tkdana@gmail.com
    from: 'ktdhanasekar@gmail.com'
    smarthost: smtp.gmail.com:587
    auth_username: 'ktdhanasekar@gmail.com'
    auth_identity: 'ktdhanasekar@gmail.com'
    auth_password: '***********'
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
__for Mattermost Notification__
```
vim /usr/local/bin/alertmanager/alertmanager.yml
```
```
global:
  resolve_timeout: 5m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 24h
  receiver: 'mattermost'
route:
  receiver: 'mattermost'
receivers:
  - name: 'mattermost'
    slack_configs:/webhook_configs:
      - api_url: 'https://convo.darthcentral.in/hooks/rtusjdiamjb4pgi3jwhii58gbr'
        send_resolved: true 
```

:blue_square: __Alert Manager Rules config__

```
mkdir /etc/prometheus/rules
```
```
vim /etc/prometheus/rules/alert-rules.yml

groups:
- name: alert-rules
  rules:
  - alert: ExporterDown
    expr: up == 0
    for: 2m
    labels:
      severity: critical
    annotations:
      description: 'Metrics exporter service for {{ $labels.job }} running on {{ $labels.instance }} has been down for more than 5 minutes.'
      summary: 'Exporter down (instance {{ $labels.instance }})'

  - alert: HostOutOfDiskSpace
    expr: (node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes < 15 and ON (instance, device, mountpoint) node_filesystem_readonly == 0
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host out of disk space (instance {{ $labels.instance }})
      description: "Disk is almost full (< 15% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

  - alert: HostOutOfMemory
    expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100 < 15
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host out of memory (instance {{ $labels.instance }})
      description: "Node memory is filling up (< 15% left)\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"

  - alert: HostHighCpuLoad
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: Host high CPU load (instance {{ $labels.instance }})
      description: "CPU load is > 85%\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
      
  - alert: SITE_DOWN
    expr: probe_http_status_code != 200
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "HTTP status code is not 200 for {{ $labels.instance }}"
      description: "Target {{ $labels.instance }} returned HTTP status code {{ $value }}. Expected 200."
      
  - alert: BlackboxSslCertificateWillExpireSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 3
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Blackbox SSL certificate will expire soon (instance {\{ $labels.instance }})
      description: "SSL certificate expires in 3 days\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

:blue_square: __Prometheus Queries for customised dashboard__

ALL RAM
```
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```
ALL CPU
```
100 * (1 - avg by(job) (rate(node_cpu_seconds_total{mode="idle"}[5m])))
```
ALL HDD
```
100 - ((node_filesystem_avail_bytes {mountpoint="/",fstype!="rootfs"} * 100) / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"})
```

:blue_square: __amtool and promtool__ \
To check the validation of alertmanager.yml
```
/usr/local/bin/alertmanager/amtool check-config /usr/local/bin/alertmanager/alertmanager.yml
```
To check for alert rules validation
```
promtool check rules /etc/prometheus/rules/alert-rules.yml
```
finally restart and check the status of the services
```
systemctl restart prometheus && systemctl restart node_exporter && systemctl restart alertmanager && systemctl restart grafana-server
```
```
systemctl status prometheus --no-pager && systemctl status node_exporter --no-pager && systemctl status alertmanager --no-pager && systemctl status grafana-server --no-pager
```

