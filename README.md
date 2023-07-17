# install-SSL-odoo16-on-ubuntu (EC2 ubuntu instance)

## 1. First we need install Certbot:
  `sudo apt update`
  
  `sudo apt install certbot`
  Then run command below to create a configuration SSL certificate for your server, example is `td.com:8069` is my odoo domain 

  `sudo certbot certonly --standalone -d td.com`

  After done this task let check your certificate path and private key path using
  
  `sudo certbot certificates`
  Take note for your certificate path and private key path for configuration nginx later.
  
## 2. Install nginx and configuration for odoo:
   `sudo apt install nginx`.
   
   Then create a configuration for your domain:
   
   
   `sudo nano /etc/nginx/sites-available/odoo.con`

   ```upstream odoo {
server 127.0.0.1:8069;
}

server {
    if ($host = td.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


listen 80;
server_name td.com;
return 301 https://$host$request_uri;


}

server {
listen 443 ssl;
server_name td.com;


access_log /var/log/nginx/odoo.access.log;
error_log /var/log/nginx/odoo.error.log;
    ssl_certificate /etc/letsencrypt/live/td.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/td.com/privkey.pem; # managed by Certbot


location / {
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_redirect off;
    proxy_pass http://odoo;
}

location ~* /web/static/ {
    proxy_cache_valid 200 90m;
    proxy_buffering on;
    expires 864000;
    proxy_pass http://odoo;
}

gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
gzip on; ```

Then `Ctrl + X` -> `Y` -> `Enter` for save and run this to check the syntax of configuration file: `sudo nginx -t`

If it ok then let restart the nginx for apply the configuration:
`sudo systemctl restart nginx`

