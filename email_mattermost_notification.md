```
global:
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alerts@example.com'
  smtp_auth_username: 'alerts@example.com'
  smtp_auth_password: 'your-smtp-password'

route:
  receiver: 'email-and-mattermost'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h

receivers:
  - name: 'email-and-mattermost'
    email_configs:
      - to: 'alerts-team@example.com'
        send_resolved: true
    webhook_configs:
      - url: 'https://mattermost.example.com/hooks/your-webhook-id'
        send_resolved: true
```

