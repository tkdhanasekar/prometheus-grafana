
## How to run grafana in reverse proxy

Install NGINX
```
sudo apt install nginx -y
```
Configure Grafana for Reverse Proxy
```
sudo vim /etc/grafana/grafana.ini
```
```
[server]
# Protocol (http, https, socket)
protocol = http

# The ip address to bind to, empty will bind to all interfaces
http_addr = 127.0.0.1

# The http port to use
http_port = 3000

# The public facing domain name used to access grafana from a browser
domain = yourdomain.com

# Set to true if you are using a reverse proxy
serve_from_sub_path = true

# The full public facing url
root_url = %(protocol)s://%(domain)s/
```
:wq!

restart Grafana
```
sudo systemctl restart grafana-server
```
Configure NGINX as Reverse Proxy
Create a new NGINX configuration file
```
sudo vim /etc/nginx/sites-available/grafana
```
```
server {
    listen 80;
    server_name yourdomain.com;  # Change to your domain or IP

    location / {
        proxy_pass http://127.0.0.1:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Enable the config
```
sudo ln -s /etc/nginx/sites-available/grafana /etc/nginx/sites-enabled/
```
Test and reload
```
sudo nginx -t
```
```
sudo systemctl reload nginx
```
Secure with Let's Encrypt SSL
```
sudo apt install certbot python3-certbot-nginx -y
```
```
sudo certbot --nginx -d yourdomain.com
```



