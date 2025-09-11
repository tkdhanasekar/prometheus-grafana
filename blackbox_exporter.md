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
```
```
modules:
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: [200]  # Defaults to 2xx
      method: GET
      headers:
        User-Agent: "Blackbox"
      fail_if_ssl: false
      fail_if_not_ssl: false
      follow_redirects: true
      preferred_ip_protocol: "ip4"

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

ExecStart=/root/blackbox_exporter-0.27.0.linux-amd64/blackbox_exporter --config.file=/root/blackbox_exporter-0.27.0.linux-amd64/monitor_website.yml

[Install]
WantedBy=multi-user.target
```
daemon-reload, start, enable the service
```
systemctl daemon-reload && systemctl start blackbox && systemctl enable blackbox && systemctl status blackbox --no-pager
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
        replacement: blackbox_server_ip:9115
```
```
systemctl restart prometheus && systemctl restart grafana-server        
```
To view the blackbox metrics
http://server_ip:9115/

To check website is working or not working
- In grafana dashboard
- create new dashboard > add new panel
```
probe_http_status_code
```
- execute the query in promQL
- change the visualization to table in top right of the grafana dashboard,
- set panel title,
- set value mapping 
  - value 200 display text OK set color
  - vaule range 0-199 text NOT_OK set color
  - vaule range 201-600 text NOT_OK set color,
- then set cell display mode to color text,
- then in promQL Format change timeseries to table and set Type to instant,
- and then from transform filter by name , disable time, name, job
- apply and save

To check for SSL site
- execute the query in promQL
```
sum by(instance) (probe_http_ssl)
```
- change the visualization to table in top right of the grafana dashboard,
- set panel title,
- set value mapping 
  - value 1 display text SSL set color
  - value 0 display text NO_SSL set color
- then set cell display mode to color text,
- then in promQL Format change timeseries to table and set Type to instant,
- and then from transform filter by name , disable time name job
- apply and save


Alert rule that triggers when the probe_http_status_code is not equal to 200,
```
groups:
- name: blackbox-alerts
  rules:
  - alert: HTTPStatusNot200
    expr: probe_http_status_code != 200
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "HTTP status code is not 200 for {{ $labels.instance }}"
      description: "Target {{ $labels.instance }} returned HTTP status code {{ $value }}. Expected 200."
```
Alert rule for SSL certificates that are about to expire within the next 30 days
```
groups:
- name: ssl-certificate-alerts
  rules:
  - alert: SSLCertificateExpiringSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 30 * 24 * 60 * 60
    for: 1h
    labels:
      severity: warning
    annotations:
      summary: "SSL certificate for {{ $labels.instance }} expires soon"
      description: "The SSL certificate for {{ $labels.instance }} will expire in less than 30 days (on {{ probe_ssl_earliest_cert_expiry | humanizeTimestamp }})."
```
Alert rule for SSL certificates that are about to expire within days
```
- alert: BlackboxSslCertificateWillExpireSoon
  expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 3
  for: 0m
  labels:
    severity: critical
  annotations:
    summary: Blackbox SSL certificate will expire soon (instance {\{ $labels.instance }})
    description: "SSL certificate expires in 3 days\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```


