:red_square: __Prometheus Installation__ \
:red_square: __Grafana Installation__ \
:red_square: __Node Exporter Installation__ \
:red_square: __Node Exporter Installation in client Machines__ \
:red_square: __Alert Manager Installation__ \
:red_square: __scrape config__ \
:red_square: __SMTP config for Alerts__ \
:red_square: __Alert Manager rules__ 



:blue_square: __Prometheus Installation__

```
apt update -y && apt upgrade -y
```

Download the latest prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.25.2/prometheus-2.25.2.linux-amd64.tar.gz
```
```
tar -xvf prometheus-2.25.2.linux-amd64.tar.gz
```
rename the prometheus-2.25.2.linux-amd64 to prometheus-files
```
mv prometheus-2.25.2.linux-amd64 prometheus-files
```
Add a Prometheus user
```
useradd --no-create-home --shell /bin/false prometheus
```
create necessary directories
```
mkdir /etc/prometheus
mkdir /var/lib/prometheus
groupadd prometheus
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
Move the consoles and console_libraries directories from prometheus-files to /etc/prometheus folder and change the ownership to prometheus user
```
cp -r prometheus-files/consoles /etc/prometheus
cp -r prometheus-files/console_libraries /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/consoles
chown -R prometheus:prometheus /etc/prometheus/console_libraries
```
Setup Prometheus Configuration
Create the prometheus.yml file.
```
vim /etc/prometheus/prometheus.yml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'prometheus_server'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.104.47.222:9100']

  - job_name: 'client_1'
    scrape_interval: 5s
    static_configs:
      - targets: ['139.162.56.155:9100']
      
  - job_name: 'client_2'
    scrape_interval: 5s
    static_configs:
      - targets: ['172.105.123.69:9100']
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
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl status prometheus
```
access the prometheus UI on 9090 port \
http://xx.xx.xx.xx:9090/graph

:blue_square: __Grafana Installation__
update the server
```
apt update -y
```
download the grafana GPG key
```
wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null
```
install Grafana APT repository
```
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
update the server
```
apt update
```
install the grafana package
```
apt install grafana -y
```
start the grafana server
```
systemctl start grafana-server.service
```
enable the grafana server
```
systemctl enable grafana-server.service
```
check the status of grafana server
```
systemctl status grafana-server.service
```
check the version of grafana server
```
grafana-server -v
```
to login to grafana dashboard \
http://ip_of_server:3000/login


:blue_square: __Node Exporter Installation__
download the source
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz
```
```
tar -xvf node_exporter-1.1.2.linux-amd64.tar.gz
```
move the folder to the specified location
```
mv node_exporter-1.1.2.linux-amd64/node_exporter /usr/local/bin/
```
add the user
```
useradd -rs /bin/false node_exporter
```
create the group
```
groupadd node_exporter
```
create the node_exporter service file
```
vim /etc/systemd/system/node_exporter.service

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
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
systemctl status node_exporter
```
in the browser \
http://server_IP:9100/metrics

:blue_square: __Node Exporter Installation in Client Machines__
install node_exporter in all client machines

:blue_square: __Add Data sources__
add data sources \
add node exporter full 1860 id for dashboard


:blue_square: __Alert Manager Installation in Prometheus Server__

update the server
```
apt update -y
```
Download the alertmanager
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
tar -xvf alertmanager-0.23.0.linux-amd64.tar.gz
cd alertmanager-0.23.0.linux-amd64/
cp -r . /usr/local/bin/alertmanager
```
create the alertmanager service file
```
vim /etc/systemd/system/alertmanager.service
[Unit]
Description=Prometheus Alert Manager Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager \
    --config.file=/usr/local/bin/alertmanager/alertmanager.yml \
    --cluster.advertise-address="172.104.47.222:9093"

[Install]
WantedBy=multi-user.target
```
check the alertmanager config with amtool
```
/usr/local/bin/alertmanager/amtool check-config /usr/local/bin/alertmanager/alertmanager.yml
```
reload the daemon , start , enable and check the status of alertmanager server
```
systemctl daemon-reload
systemctl start alertmanager.service
systemctl enable alertmanager.service
systemctl status alertmanager.service
```

in browser \
http://server_ip:9093

:blue_square: __Add alert rules location__

Add alertmanager parameters in /etc/prometheus/prometheus.yml:
```

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "/etc/prometheus/rules/*.yml"
```

:blue_square: __SMTP Config for Alert Manager__
```
vim /usr/local/bin/alertmanager/alertmanager.yml
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
```

:blue_square: __customised dashboard__
```
https://github.com/tkdhanasekar/prometheus-grafana/blob/main/16.All_Linx_CPU.json
https://github.com/tkdhanasekar/prometheus-grafana/blob/main/17.All_Linux_RAM.json
https://github.com/tkdhanasekar/prometheus-grafana/blob/main/18.All_Linux_HDD.json
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
