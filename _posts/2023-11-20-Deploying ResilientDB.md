---
layout: article
title: Deploying ResilientDB
author: Apratim Shukla
tags: NexRes Deployment ResVault ResilientDBGraphQL
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/code_close_up.jpeg

---

This tutorial assumes that you have setup ResilientDB by following this [blog](https://blog.resilientdb.com/2023/09/21/ResVault.html). In order to get started with deploying ResilientDB and the extended applications on cloud you will need to follow the steps below.

# Prerequisites
### Create a Cloud instance
You can create a cloud instance on either AWS [(EC2)](https://console.aws.amazon.com/ec2/), GCP [(Compute Engine)](https://console.cloud.google.com/compute/instances?_ga=2.30596543.1077298781.1700583534-792349637.1700583534), or choose any other Virtual Private Server [(VPS)](https://en.wikipedia.org/wiki/Virtual_private_server). Ensure that you select the image as Ubuntu 22.04.

### Setup ResilientDB and other repositories
- Clone and Install the dependencies for ResilientDB, CROW HTTP Server, and GraphQL by following this [blog]((https://blog.resilientdb.com/2023/09/21/ResVault.html).

### Purchase a domain and setup Cloudflare DNS
- Go to [Namecheap](https://www.namecheap.com/) and purchase a domain. Then, once you have registered a domain click on Manage option and under DNS select custom DNS.

- Then go to [Cloudflare](https://www.cloudflare.com/en-in/) and create an account. Then, proceed to add the domain that you just registered. Add the two nameservers given by Cloudflare in the Namecheap Custom DNS. (This change might take a while to reflect)

- Now, navigate to the DNS management page on the Cloudflare Dashboard and create the following five subdomain records:
  * An A record with name **crow** and IPv4 address as the external IP of your cloud instance.
  * An A record with name **cloud** and IPv4 address as the external IP of your cloud instance.
  * An A record with name **explorer** and IPv4 address as the external IP of your cloud instance.
  * An A record with name **prometheus** and IPv4 address as the external IP of your cloud instance.
  * An A record with name **monitoring** and IPv4 address as the external IP of your cloud instance.

### Setup Nginx
- SSH into your cloud instance and install Nginx with the following command:
  > sudo apt update

  > sudo apt upgrade

  > sudo apt install nginx

- Then, start and enable Nginx as a service:
  > sudo systemctl start nginx

  > sudo systemctl enable nginx

- The next step is to create a configuration file, we are going to name it **conf** but you can name it whatever you want:

  > sudo nano /etc/nginx/sites-available/conf

- Then, copy and paste the following configuration inside the file, make sure to edit *yourdomain.com* with your domain:

```
For the Flask app
server {
   listen 80;
   server_name cloud.yourdomain.com;

   location / {
       proxy_pass http://127.0.0.1:8000;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
   }
}

# For the Vite app
server {
   listen 80;
   server_name explorer.yourdomain.com;

   location / {
       proxy_pass http://127.0.0.1:8080;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
   }

   location /block {
       proxy_pass http://127.0.0.1:8080/;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
   }

   location /transactions {
       proxy_pass http://127.0.0.1:8080/;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
   }

   location /api/ {
       proxy_pass http://localhost:18000/;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host $host;
       proxy_cache_bypass $http_upgrade;
       proxy_redirect off;
   }
}

# For the CROW HTTP server
server {
    listen 80;
    server_name crow.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:18000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # CORS headers
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept' always;
    }
   location /blockupdatelistener {
        # Set up the proxy pass to your WebSocket server
        proxy_pass http://127.0.0.1:18000;

        # Upgrade headers for WebSocket
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Set WebSocket specific headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # CORS headers
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept' always;

        # Handle WebSocket specific "OPTIONS" request
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept';
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            add_header 'Content-Length' 0;
            return 204;
        }
    }
}

# For Prometheus
server {
   listen 80;
   server_name prometheus.yourdomain.com;

   location / {
       proxy_pass http://127.0.0.1:9090;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
   }

   access_log /var/log/nginx/prometheus.access.log;
   error_log /var/log/nginx/prometheus.error.log;
}

# For Grafana
# this is required to proxy Grafana Live WebSocket connections.
map $http_upgrade $connection_upgrade {
 default upgrade;
 '' close;
}

upstream grafana {
 server localhost:3000;
}

server {
  listen 80;
  server_name monitoring.yourdomain.com;

  location / {
    proxy_set_header Host $http_host;
    proxy_pass http://grafana;
  }

# Proxy Grafana Live WebSocket connections.
  location /api/live/ {
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header Host $http_host;
    proxy_pass http://grafana;
  }
}
```

- Save the configuration file by pressing ctrl+o and then exit the editor.

- Enable the site configuration by running the following command:
  > sudo ln -s /etc/nginx/sites-available/conf /etc/nginx/sites-enabled/

- Check for syntax errors using the command below:
  > sudo nginx -t

- If everything is fine then restart Nginx to view the changes:
  > sudo systemctl restart nginx

Now, you can navigate to the different subdomains via your web browser and view the different services. 
