# NGINX Config

## Default Site

HTTP (ie: the IP) found at `/var/www/html/`
HTTPS found at `/usr/share/nginx/html/index.html`

## Reverse Proxy

Create a file at `/etc/nginx/sites-available/app-name.conf`.

Edit the config: `nano /etc/nginx/sites-available/app-name.conf`

```
server {
    listen 80;
    listen [::]:80;
    server_name hostname.com, www.hostname.com;
    
    # SSL configuration (using LetsEncrypt instructions below)
    listen 443 default_server ssl;
    ssl_certificate     /etc/letsencrypt/live/hostname.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/hostname.com/privkey.pem;

    # subdomain example
    server_name subdomain.hostname.com;

    # Default path for the hostname
    location / {
      proxy_set_header   X-Forwarded-For $remote_addr;
      proxy_set_header   Host $http_host;
      proxy_pass         http://0.0.0.0:3000/;

      proxy_set_header   Upgrade $http_upgrade; # to enable WebSockets
      proxy_set_header   Connection "upgrade"; # to enable WebSockets
    }

    # Redirecting path to app
    location /app-name/ {
        proxy_set_header   X-Forwarded-For $remote_addr;
        proxy_set_header   Host $http_host;
        proxy_pass         http://0.0.0.0:5000/;
    }
}
```

Create a symlink to enable the app:

```
sudo ln -s /etc/nginx/sites-available/app-name.conf /etc/nginx/sites-enabled/app-name.conf
```

Start NGINX or reload the configuration if service already running, then check the status:

```
sudo systemctl start nginx
sudo systemctl reload nginx
sudo systemctl status nginx
```

## SSL Certificate

First, update snap:

```
sudo snap install core; sudo snap refresh core
```

Install certbot:

```
sudo snap install --classic certbot
```

Configure the default virtual host:

```
nano /etc/nginx/conf.d/default.conf

server {
    server_name  hostname.com www.hostname.com;
}
```

Run certbot (need to do it for each subdomain):

```
certbot --nginx --redirect -d hostname.com -d www.hostname.com -m admin@hostname.com --agree-tos
```

## Running Node as a Process

https://github.com/foreversd/forever

```
# Start an app
forever start script.js

# List all running apps
forever list

# Stop an app
forever stop <index|appId>

# View app logs
forever logs <index>
```

### Force Killing the Process

```
sudo kill -9 $(sudo lsof -t -i:<PORT_NUMBER>)
```

## Checking Services

To list all services:

```
service --status-all
```

To check status of a single service:

```
systemctl status <service_name>
```

### Restarting NGINX

```
sudo systemctl reload nginx
```

## Updating Ubuntu

```
sudo apt update && sudo apt upgrade -y
```
