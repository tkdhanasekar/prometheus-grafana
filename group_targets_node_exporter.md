## Group Targets by Client for node_exporter

```
vim /etc/prometheus/prometheus.yml
```
```
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['client1-node1:9100', 'client1-node2:9100']
        labels:
          client: 'client1'
          group: 'web'

      - targets: ['client2-node1:9100']
        labels:
          client: 'client2'
          group: 'db'
```
## Define Alerting Rules per Client
```
groups:
  - name: node_alerts
    rules:
      - alert: HighCPUUsage
        expr: rate(node_cpu_seconds_total{mode="user"}[5m]) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "Client: {{ $labels.client }} - CPU usage is high"

      # You can also scope alerts only to a specific client:
      - alert: Client1NodeDown
        expr: up{client="client1"} == 0
        for: 1m
        labels:
          severity: critical
          client: client1
        annotations:
          summary: "Client1 Node Down: {{ $labels.instance }}"
```

## Route Alerts by Client in Alertmanager
```
route:
  group_by: ['alertname', 'client']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'default-receiver'

  routes:
    - match:
        client: 'client1'
      receiver: 'client1-email'

    - match:
        client: 'client2'
      receiver: 'client2-slack'

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'default@example.com'

  - name: 'client1-email'
    email_configs:
      - to: 'alerts-client1@example.com'

  - name: 'client2-slack'
    slack_configs:
      - channel: '#client2-alerts'
        send_resolved: true
```

