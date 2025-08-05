:red_square: __Setup of Prometheus Alertmanager in Ubuntu__

update the server
```
# apt update -y
```
Download the alertmanager
```
# wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
# tar -xvf alertmanager-0.28.1.linux-amd64.tar.gz
# cd alertmanager-0.28.1.linux-amd64/
# cp -r . /usr/local/bin/alertmanager
```
create the alertmanager service file
```
# vim /etc/systemd/system/alertmanager.service
[Unit]
Description=Prometheus Alert Manager Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager/alertmanager \
    --config.file=/usr/local/bin/alertmanager/alertmanager.yml \
    --cluster.advertise-address="xx.xx.xx.xx:9093"

[Install]
WantedBy=multi-user.target

:wq! 
save and exit , where xx.xx.xx.xx is prometheus alert manager server ip
```

check the alertmanager config with amtool
```
# /usr/local/bin/alertmanager/amtool check-config /usr/local/bin/alertmanager/alertmanager.yml
```
reload the daemon , start , enable and check the status of alertmanager server
```
# systemctl daemon-reload
# systemctl start alertmanager.service
# systemctl enable alertmanager.service
# systemctl status alertmanager.service
```
in browser
http://server_ip:9093 \
http://server_ip:9090/alerts
