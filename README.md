
2. [Let's Enctypt (Certbot) - Quick Installation](#Secure-NGINX-application-with-Let's-Encrypt-on-Ubuntu-18.04---Quick-Installation)
2. [Let's Enctypt - (Certbot)- Complete Installation](#Secure-NGINX-application-with-Let's-Encrypt-on-Ubuntu-18.04)


### Secure NGINX application with Let's Encrypt on Ubuntu 18.04 - Quick Installation

Domain Name : http://try_domain.com/


Step 1 — Installing Certbot

anup@megatron:~$ sudo apt-get update

anup@megatron:~$ sudo apt-get install software-properties-common

anup@megatron:~$ sudo add-apt-repository universe

anup@megatron:~$ sudo add-apt-repository ppa:certbot/certbot

anup@megatron:~$ sudo apt-get install certbot python3-certbot-nginx


Step 2 — Running Certbot

Method one  :  Certbot will edit NGINX configuration automatically to serve it, turning on HTTPS access in a single step

anup@megatron:~$ sudo certbot --nginx 


Method Two : Get certificates only where to make the changes to nginx configuration by hand 

anup@megatron:~$ sudo certbot certonly --nginx


Step 3 — Find your certificates

anup@megatron:~$ sudo ls -ltr /etc/letsencrypt/live/try_domain.com

Check cerificate details

https://www.sslshopper.com/ssl-checker.html


Step 4 — Handling Certbot Automatic Renewals

anup@megatron:~$ cd /etc/crontab

anup@megatron:~$ sudo nano /etc/cron.d/certbot

0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q$

anup@megatron:~$ sudo certbot renew --dry-run



### Secure NGINX application with Let's Encrypt on Ubuntu 18.04

`Domain Name : http://try_domain.com/`


#### Install Nginx on Ubuntu 18.04

`anup@megatron:~$ clear`

`anup@megatron:~$ sudo apt update`

`anup@megatron:~$ sudo nginx -v`

`anup@megatron:~$ sudo systemctl status nginx`

`anup@megatron:~$ ifconfig`


#### Configuring firewall for NGINX

`anup@megatron:~$ sudo ufw allow 'Nginx Full'`

`anup@megatron:~$ sudo ufw status`


#### Install Certbot

`anup@megatron:~$ sudo apt update`

`anup@megatron:~$ sudo apt install certbot`


#### Generate Strong Dh (Diffie-Hellman) Group

`anup@megatron:~$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048`


#### Obtaining a Let’s Encrypt SSL certificate

`anup@megatron:~$ sudo mkdir -p /var/lib/letsencrypt/.well-known`

`anup@megatron:~$ sudo chgrp www-data /var/lib/letsencrypt`

`anup@megatron:~$ sudo chmod g+s /var/lib/letsencrypt`


#### Create first snippet, "letsencrypt.conf" to to avoid duplicating code which we’re going to include in all our NGINX server block files

`anup@megatron:~$ sudo nano /etc/nginx/snippets/letsencrypt.conf`

```
location ^~ /.well-known/acme-challenge/ {

    allow all;

    root /var/lib/letsencrypt/;

    default_type "text/plain";

    try_files $uri =404;

}
```


#### Create second snippet, "ssl.conf" to to avoid duplicating code which we’re going to include in all our NGINX server block files

`anup@megatron:~$ sudo nano /etc/nginx/snippets/ssl.conf`

```

ssl_dhparam /etc/ssl/certs/dhparam.pem;

ssl_session_timeout 1d;

ssl_session_cache shared:SSL:50m;

ssl_session_tickets off;

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-A$

ssl_prefer_server_ciphers on;

ssl_stapling on;

ssl_stapling_verify on;

resolver 8.8.8.8 8.8.4.4 valid=300s;

resolver_timeout 30s;

add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload";

add_header X-Frame-Options SAMEORIGIN;

add_header X-Content-Type-Options nosniff;

```


#### Include the letsencrypt.conf snippet to the domain server block

`anup@megatron:~$ sudo nano /etc/nginx/sites-available/try_domain.com.conf`

```
server {

    listen 80;

    server_name www.try_domain.com try_domain.com;

    include snippets/letsencrypt.conf;

}
```


#### Create a symbolic link and restart NGINX service for the changes to take effect

`anup@megatron:~$ sudo ln -s /etc/nginx/sites-available/try_domain.com.conf /etc/nginx/sites-enabled/`

`anup@megatron:~$ sudo nginx -t`

`anup@megatron:~$ sudo systemctl restart nginx`

`anup@megatron:~$ sudo systemctl status nginx`


#### Run certbot with the webroot plugin and obtain the SSL certificate files

`anup@megatron:~$ sudo certbot certonly --agree-tos --email uniqs.anup@gmail.com --webroot -w /var/lib/letsencrypt/ -d try_domain.com -d try_domain.com`


#### Check cerificate

`https://www.sslshopper.com/ssl-checker.html`


#### Edit domain server block

`anup@megatron:~$ sudo nano /etc/nginx/sites-available/try_domain.com.conf`

```
server {

listen 80;

server_name www.try_domain.com try_domain.com;

include snippets/letsencrypt.conf;

return 301 https://$host$request_uri;

}

server {

listen 443 ssl http2;

server_name www.try_domain.com;

ssl_certificate /etc/letsencrypt/live/try_domain.com/fullchain.pem;

ssl_certificate_key /etc/letsencrypt/live/try_domain.com/privkey.pem;

ssl_trusted_certificate /etc/letsencrypt/live/try_domain.com/chain.pem;

include snippets/ssl.conf;

include snippets/letsencrypt.conf;

return 301 https://try_domain.com$request_uri;

}

server {

listen 443 ssl http2;

server_name chat.dreamorbit.com;

ssl_certificate /etc/letsencrypt/live/try_domain.com/fullchain.pem;

ssl_certificate_key /etc/letsencrypt/live/try_domain.com/privkey.pem;

ssl_trusted_certificate /etc/letsencrypt/live/try_domain.com/chain.pem;

include snippets/ssl.conf;

include snippets/letsencrypt.conf;

. . . other code

}
```


#### Reload NGINX service for the changes to take effect

`anup@megatron:~$ sudo nginx -t`

`anup@megatron:~$ sudo systemctl reload nginx`

`anup@megatron:~$ sudo systemctl status nginx`


#### Auto-renewing Let’s Encrypt SSL certificate

`anup@megatron:~$ sudo nano /etc/cron.d/certbot`

`0 */12 * * * root test -x /usr/bin/certbot -a ! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q$`


#### To test the renewal process, use "certbot --dry-run"

`anup@megatron:~$ sudo certbot renew --dry-run`
