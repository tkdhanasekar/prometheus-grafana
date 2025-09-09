## grouping website internal and external website and ssl status 
add the scrape config
```
vim /etc/prometheus/prometheus.yml
```
```
- job_name: 'blackbox-internal'
    metrics_path: /probe
    params:
      module: [http_2xx_example]
    static_configs:
      - targets:
          - https://site1.in
          - https://site2.in
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: job
        replacement: blackbox-internal
      - target_label: site_type
        replacement: internal
      - target_label: __address__
        replacement: blackbox_server_ip:9115

  - job_name: 'blackbox-external'
    metrics_path: /probe
    params:
      module: [http_2xx_example]
    static_configs:
      - targets:
        - https://site3.com
        - https://site4.com

    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: job
        replacement: blackbox-external
      - target_label: site_type
        replacement: external
      - target_label: __address__
        replacement: blackbox_server_ip:9115
```
        
Panel 1: Internal website status
```
probe_http_status_code{site_type="internal"}
```
Panel 2: External website status
```
probe_http_status_code{site_type="external"}
```
Panel 3: Internal site SSL status
```
probe_http_ssl{site_type="internal"}
```
Panel 4: External site SSL status
```
probe_http_ssl{site_type="external"}
```
