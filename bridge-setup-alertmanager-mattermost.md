install docker
```
apt install docker.io
```
start enable and check status of docker
```
systemctl start docker && systemctl enable docker && systemctl status docker --no-pager
```
create folder 
```
mkdir bridge-setup && cd bridge-setup
```
create the Dockerfile
```
vim Dockerfile
```
```
FROM python:3.12-slim or 3.10-slim
WORKDIR /app
COPY alertmanager_to_mattermost.py .
RUN pip install flask requests 
EXPOSE 7777
CMD ["python", "alertmanager_to_mattermost.py"]
```
:wq!

create the python program to setup bridge between prometheus alertmanager and mattermost
and give the MATTERMOST webhook_url in the python program
```
vim alertmanager_to_mattermost.py
```
```
from flask import Flask, request, jsonify
import requests
import os

app = Flask(__name__)

# Set your Mattermost webhook URL (or load from env)
MATTERMOST_WEBHOOK_URL = os.getenv("MATTERMOST_WEBHOOK_URL", "https://webhook_rul/***********")

@app.route("/alert", methods=["POST"])
def alertmanager_webhook():
    data = request.get_json()

    if not data or "alerts" not in data:
        return jsonify({"error": "Invalid payload"}), 400

    alerts = data["alerts"]
    message_lines = []

    for alert in alerts:
        status = alert.get("status", "unknown").upper()
        labels = alert.get("labels", {})
        annotations = alert.get("annotations", {})

        alertname = labels.get("alertname", "Unnamed Alert")
        instance = labels.get("instance", "unknown")
        severity = labels.get("severity", "unknown")

        description = annotations.get("description", "")
        summary = annotations.get("summary", "")

        msg = f"### [{status}] {alertname}\n"
        msg += f"- **Instance**: `{instance}`\n"
        msg += f"- **Severity**: `{severity}`\n"
        if summary:
            msg += f"- **Summary**: {summary}\n"
        if description:
            msg += f"- **Description**: {description}\n"

        message_lines.append(msg)

    payload = {
        "text": "\n---\n".join(message_lines)
    }

    response = requests.post(MATTERMOST_WEBHOOK_URL, json=payload)

    if response.status_code != 200:
        return jsonify({"error": "Failed to send to Mattermost", "details": response.text}), 500

    return jsonify({"status": "ok"}), 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=7777)
```
:wq!

Build the docker image
```
docker build -t alertmatter-python .
```
create docker container
```
docker run -d -p 7777:7777 -e MATTERMOST_WEBHOOK_URL=https://webhook_url/hooks/******** alertmatter-python
```
give the webhook_config url as http://<bridge-host:7777>/alert in the alertmanager.yml file
```
vim /usr/local/bin/alertmanager/alertmanager.yml
```
```
receivers:
  - name: mattermost-webhook
    webhook_configs:
      - url: http://<bridge-host>:7777/alert
route:
  receiver: mattermost-webhook
```
:wq!
restart the alertmanager service
```
systemctl restart alertmanager
```

