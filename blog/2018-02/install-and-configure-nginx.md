---
title: How to install and configure NGINX with Octopus Deploy
description: Learn how to create a repeatable deployment process for installing and configuring NGINX.
author: shane.gill@octopus.com
visibility: public
published: 2018-02-02
metaImage: install-and-configure-nginx/metaimage-nginx.png
bannerImage: install-and-configure-nginx/blogimage-nginx.png
tags:
 - DevOps
---

Hello deployers!  Over the past few days I have learned a lot about configuring NGINX. I have configured a simple reverse proxy for two web applications and hope to never do it again. For the sake of my future self I have setup a project in Octopus to do the work. A configuration change, certificate expiry or deployment to another machine can be completed at the push of a button, rather than trying to remember all of the tweaking I have done over the past days. Here's how it works:
Hello deployers!  Over the past few days I have learned a lot about configuring NGINX. I have configured a simple reverse proxy for two web applications and hope to never do it again. For the sake of my future self I have setup a project in Octopus to do the work. A configuration change, certificate expiry or deployment to another machine can be completed at the push of a button, rather than trying to remember all of the tweaking I have done over the past days. Here's how it works:

![NGINX deployment process](install-and-configure-nginx/nginx-deployment-process.png)

## Install NGINX
To use NGINX we must install NGINX.  A simple bash script ensures that NGINX is installed:

```bash
sudo apt-get install --assume-yes nginx
```

## Install site configurations
There are two configuration files that instruct NGINX how to forward requests to our web applications. They look something like:

```no-highlight
server {
    listen 80; listen [::]:80;
    server_name somewhere.octopus.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;  listen [::]:443 ssl http2;
    server_name somewhere.octopus.com;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl on;
    ssl_certificate      /etc/ssl/octopus.crt;
    ssl_certificate_key  /etc/ssl/octopus.key;
    ssl_session_tickets off;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA;
    ssl_prefer_server_ciphers on;

    http2_idle_timeout 5m; # up from 3m default

    location / {
        proxy_pass http://unix:/var/discourse/shared/standalone/nginx.http.sock:;
        proxy_set_header Host $http_host;
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

The files live in source control and get packaged up to be used by Octopus. This step copies the files to their destination in the `sites-available` NGINX configuration directory, and then creates a symbolic link in the `sites-enabled` directory. When NGINX start, it loads the configuration from the `sites-enabled` directory and starts forwarding requests to those sites. There is a post-deployment bash script that looks like:

```bash
sudo rm /etc/nginx/sites-enabled/*
sudo rm /etc/nginx/sites-available/*

sudo cp somewhere.nginx /etc/nginx/sites-available/somewhere
sudo ln -s /etc/nginx/sites-available/somewhere /etc/nginx/sites-enabled/somewhere
```

## Install SSL certificate
Octopus has first class support for SSL certificates. This step copies the SSL certificate and key file to the location specified in the site configuration above (/etc/ssl). NGINX is expecting a PEM file for the key and a PEM file for the certificate. The certificate file must contain the entire certificate chain, that is the primary certificate and any intermediate certificates. To allow Octopus to provide the certificate in the correct format, I created a single PEM file with the primary certificate, intermediate certificates and private key. It looks something like:

```no-highlight
-----BEGIN CERTIFICATE-----
PRIMARY_CERTIFICATE
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
INTERMEDIATE_CERTIFICATE
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
PRIVATE_KEY
-----END RSA PRIVATE KEY-----
```

The certificate is in the Octopus certificates library. The step has a variable called `sslCert` which references the library certificate. In the step, the variable is used to create the certificate and key files and then copy them to their respective locations in the /etc/ssl directory. Note that `RawOriginal` is used to maintain the certificate chain:

```bash
KEY=$(get_octopusvariable "sslCert.PrivateKeyPem")
echo "$KEY" > ssl.key
sudo mv ssl.key /etc/ssl/octopus.key

CERT=$(get_octopusvariable "sslCert.RawOriginal")
echo "$CERT" | base64 -d > ssl.crt
sudo mv ssl.crt  /etc/ssl/octopus.crt
```

## Starting NGINX
Finally, it is time to start (or restart) NGINX. This step is a simple bash script to restart the NGINX service:

```bash
sudo service nginx restart
```

## Conclusion
With a repeatable deployment process set up, I can now make changes to the configuration files or renew the certificate without having to spend hours relearning how to configure NGINX. One of my colleagues can pick up the project and see how NGINX has been configured so they can start contributing in minutes, rather than wading through documentation or trying to reverse engineer what I had done. Best of all, if the boss asks me to configure NGINX again I can just reuse my Octopus Deploy project and spend the rest of the day at the beach. Happy deployments!
