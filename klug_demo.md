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
      - targets: ['xx.xx.xx.xx:9100']

  - job_name: 'client_1'
    scrape_interval: 5s
    static_configs:
      - targets: ['xx.xx.xx.xx:9100']
      
  - job_name: 'client_2'
    scrape_interval: 5s
    static_configs:
      - targets: ['xx.xx.xx.xx:9100']
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
download the package
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```
install Grafana APT repository
```
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
```
update the server
```
apt update -y
```
install the grafana package
```
apt-get -y install grafana
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
