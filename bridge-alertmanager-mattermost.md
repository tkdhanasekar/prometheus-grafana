```
docker pull ghcr.io/mr-karan/alertmatter:latest
```
```
docker run -d --name alertmatter ghcr.io/mr-karan/alertmatter
```
```
./bin/alertmanager-mattermost.bin --addr=alertmanager_server_ip:7777 --webhook-url=https://mattermost.corp.internal/hooks/johndoe(webhook url)
```
```
receivers:
  - name: 'mattermost-webhook'
    webhook_configs:
      - url: 'http://<your-service-address>:7777/alert?channel=<channel-name>' 
```
https://github.com/mr-karan/alertmatter
