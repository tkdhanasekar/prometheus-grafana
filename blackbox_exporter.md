download the blackbox exporter
```
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.27.0/blackbox_exporter-0.27.0.linux-amd64.tar.gz
```
extract
```
tar -xvf blackbox_exporter-0.27.0.linux-amd64.tar.gz
```
go to blackbox directory
```
cd blackbox_exporter-0.27.0.linux-amd64
```
create the file
```
vim monitor_website.yml
modules:
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: [200]  # Defaults to 2xx
      method: GET
```
create the service file for blackbox 
```
vim /etc/systemd/system/blackbox.service
```
```
[Unit]
Description = Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=root
Restart=on-failure

ExecStart=/root/blackbox_exporter-0.24.0.linux-amd64/blackbox_exporter --config.file=/root/blackbox_exporter-0.24.0.linux-amd64/monitor_website.yml

[Install]
WantedBy=multi-user.target
```
daemon-reload, start, enable the service
```
systemctl daemon-reload
systemctl start blackbox
systemctl enable blackbox
systemctl status blackbox
```

In Prometheus server
create scrape config file
```
vim /etc/prometheus/prometheus.yml
```
```
- job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx_example]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://kaniyam.com
        - http://freetamilebooks.com
        - http://google.com
        - http://youtube.com
        - http://instagram.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.122.164:9115
```
```
systemctl restart prometheus
systemctl restart grafana-server        
```
To view the blackbox metrics
http://server_ip:9115/

In grafana dashboard
create new dashboard

execute the querires for job
```
probe_http_status_code
```
```
probe_http_ssl
```
In visualizations > select > Tables
and then to value mappings and colour setting
and then set panel title

In bottom options
select format > Table 
select Type > instant

Select Transform data > select by name and give choice 



