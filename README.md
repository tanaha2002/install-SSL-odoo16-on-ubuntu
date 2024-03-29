# install-SSL-odoo16-on-ubuntu (EC2 ubuntu instance)
## Prerequisite
  Make sure you already open port `80` and `443` for your ubuntu instance
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
   
   Then create a configuration for your domain
   
   

   `sudo nano /etc/nginx/sites-available/odoo.conf`

You will need to re-config your `ssl_certificate` and `ssl_certificate_key` to your `certificate path` and `private key path`.
   

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
  gzip on;
```


Then `Ctrl + X` -> `Y` -> `Enter` for save and run this to check the syntax of configuration file: `sudo nginx -t`
Then use this command to create symlink
`sudo ln -s /etc/nginx/sites-available/td.conf /etc/nginx/sites-enabled/`


If it ok then let restart the nginx for apply the configuration:
`sudo systemctl restart nginx`


# Renew the Certbot if it expire automatic (EC2 ubuntu instance)
## Maunal:
1. `sudo systemctl stop nginx`
2. `sudo certbot renew`
3. `sudo systemctl restart nginx`

## Automation
1. `sudo systemctl stop nginx`
2. `sudo crontab -e`
3. `0 */12 * * * root test -x /usr/bin/certbot -a ! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q renew`

It will check and renew the certbot each 12 hours.
