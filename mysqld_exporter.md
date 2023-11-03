Install mysql server in target machine
```
apt update
apt install mysql-server
systemctl start mysql
systemctl enable mysql
systemctl status mysql
```
add user prometheus and add in prometheus group
```
useradd --no-create-home --shell /bin/false prometheus
groupadd --system prometheus
useradd -s /sbin/nologin --system -g prometheus prometheus
```
download the latest mysqld_exporter
```
curl -s https://api.github.com/repos/prometheus/mysqld_exporter/releases/latest | grep browser_download_url   | grep linux-amd64 | cut -d '"' -f 4   | wget -qi -
```
extract
```
tar xvf mysqld_exporter*.tar.gz
```
move to /usr/local/bin
```
mv  mysqld_exporter-*.linux-amd64/mysqld_exporter /usr/local/bin/
```
give execute permission to mysqld_exporter
```
chmod +x /usr/local/bin/mysqld_exporter
```
version check
```
mysqld_exporter  --version
```
create mysql user and database for mysqld_exporter
```
mysql -u root -p
mysql> CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY 'StrongPassword';
mysql> GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> EXIT
```
Configure database credentials
```
vim /etc/.mysqld_exporter.cnf
```
```
[client]
user=mysqld_exporter
password=StrongPassword
```
Set ownership permissions
```
chown root:prometheus /etc/.mysqld_exporter.cnf
```
Create systemd unit file
```
vi /etc/systemd/system/mysql_exporter.service
```
```
[Unit]
Description=Prometheus MySQL Exporter
After=network.target
User=prometheus
Group=prometheus

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/mysqld_exporter \
--config.my-cnf /etc/.mysqld_exporter.cnf \
--collect.global_status \
--collect.info_schema.innodb_metrics \
--collect.auto_increment.columns \
--collect.info_schema.processlist \
--collect.binlog_size \
--collect.info_schema.tablestats \
--collect.global_variables \
--collect.info_schema.query_response_time \
--collect.info_schema.userstats \
--collect.info_schema.tables \
--collect.perf_schema.tablelocks \
--collect.perf_schema.file_events \
--collect.perf_schema.eventswaits \
--collect.perf_schema.indexiowaits \
--collect.perf_schema.tableiowaits \
--collect.slave_status \
--web.listen-address=0.0.0.0:9104

[Install]
WantedBy=multi-user.target
```
reload the daemon, start and enable the service
```
systemctl daemon-reload
systemctl enable mysql_exporter
systemctl start mysql_exporter
```

In prometheus server add to scrape config file
```
vim /etc/prometheus/prometheus.yml
```
```
- job_name: 'server1_db'
    scrape_interval: 5s
    static_configs:
      - targets: ['server_ip:9104']
```
      
In alert rules add customised rule for mysqld
for mysqld_exporter down alert rule
```
vim /etc/prometheus/rules/alert-rules.yml
```
```
alertmanager rules:
- alert: MysqlDown
    expr: mysql_up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: MySQL down (instance {{ $labels.instance }})
      description: "MySQL instance is down on {{ $labels.instance }}\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

```
systemctl restart prometheus
systemctl restart grafana
systemctl restart node_exporter
systemctl restart alertmanager
```

mysqld exporter panel json file
```
https://github.com/prometheus/mysqld_exporter/blob/main/mysqld-mixin/dashboards/mysql-overview.json#L3
```














