# NGINX Config

## Reverse Proxy

Create a file at `/etc/nginx/sites-available/app-name.conf`.

Edit the config: `nano /etc/nginx/sites-available/app-name.conf`

```
server {
    listen 80;
    listen [::]:80;
    server_name hostname.com, www.hostname.com;

    # Default path for the hostname
    location / {
      proxy_set_header   X-Forwarded-For $remote_addr;
      proxy_set_header   Host $http_host;
      proxy_pass         http://0.0.0.0:3000/;
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

Run certbot:

```
certbot --nginx --redirect -d hostname.com -d www.hostname.com -m admin@hostname.com --agree-tos
```