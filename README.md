# Nginx Image for Azure App Service Linux

Nginx Image for serving Static HTML Apps or Single Page Applications(React,Angular) using Azure App Service Linux

[![Azure Deploy](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Flepskiy%2Fazure-appservice-nginx%2Fmaster%2Ftemplate.json)

### Benefits:
- Image Size < 70MB
- Easy to Use/Understand/Configure
- SSH access to App Container

You can control nginx configuration used in app container by creating  `nginx.conf` file @ `/home/site` folder

Ex: default root folder is set to `/home/site/wwwroot`. You can change it to `/home/site/wwwroot/build` by creating `nginx.conf` file @ `/home/site` with below content.

```
worker_processes auto;
pid          /var/run/nginx.pid;
daemon off;

events {
  worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log off;
    tcp_nopush     on;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen 80;
        server_name  www.example.com;
        error_log  /home/LogFiles/error.log warn;
        root   /home/site/wwwroot/build;
        index  Default.htm Default.html index.html index.htm hostingstart.html;

        # Make site accessible from http://localhost/
        server_name _;
        sendfile on;

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|mp3|html)$ {
            expires 1d;
        }

    }
}

```

## Building and Running Locally

You can build the Docker image directly from this repository and run it on your machine:

```bash
docker build -t azure-appservice-nginx .
docker run -p 80:80 -p 2222:2222 azure-appservice-nginx
```

The container exposes port `80` for HTTP traffic and `2222` for SSH access.

## Deploying with the ARM Template

The provided `template.json` and `parameters.json` files allow you to deploy the image to Azure App Service using an ARM template. After creating a resource group, run the following commands:

```bash
az group create --name <resource-group> --location <location>
az deployment group create \
  --resource-group <resource-group> \
  --template-file template.json \
  --parameters @parameters.json
```

Update `parameters.json` to set a unique `siteName` before deployment.

## Securing SSH Access

SSH is enabled for convenience, but the default image sets the root password to `Docker!`. For production deployments you should change this password, disable root login, or configure key-based authentication by customizing `sshd_config` or extending the Dockerfile.
