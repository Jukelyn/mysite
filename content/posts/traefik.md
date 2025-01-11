+++
draft = false
date = "2024-05-30"
title = "How To Setup Traefik For Reverse Proxying Using Portainer (Docker) "
description = "How to use traefik to reverse proxy."
slug = "traefik"
authors = ["Jukelyn"]
categories = ["Traefik", "Reverse Proxy", "Website"]
+++

[Jump to Updates]({{< ref "#updates" >}})
### What is traefik?

Traefik is an open-source edge router and reverse proxy that manages and routes traffic to services dynamically. It integrates with technologies like Docker and Kubernetes, automatically detects changes, supports multiple protocols (HTTP, HTTPS, TCP, WebSocket), and includes features like automatic SSL/TLS certificates from Let's Encrypt (sometimes a pain in the ass). Traefik also offers middlewares for added functionalities and extensive observability through metrics and logs, making it ideal for scalable, high-traffic microservices environments.

### Installing and using traefik as a reverse proxy (with SSL/TLS)

I only use traefik as a reverse proxy and SSL/TLS for this site. You may also know that nginx has a reverse proxy manager as well but I find traefik to be easier to use and I like the web UI more as well. I may add some middlewares in the future but at the moment, I don't feel like it. Anyways, for Docker containers I prefer using a configuration file and docker compose to spin up containers. You could also spawn containers using commands or a DockerFile (similar to the configuration file + compose method) but I prefer yaml so I will be doing that over DockerFile.


This is what my full traefik `docker-compose.yml` looks like:
```yaml
version: '3'

services:
  reverse-proxy:
    container_name: traefik
    image: traefik:v2.11.6
    ports:
      # The HTTP port
      - "80:80" # web
      - "443:443" # websecure
      # The Web UI port
      - "8081:8080"
    volumes:
      # Config file
      - /etc/traefik/traefik.yaml:/etc/traefik/traefik.yaml
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # For TLS certs not to be re-generated every time...
      - /etc/traefik/certs/acme.json:/acme.json
    networks:
      - nginx

networks:
  nginx:
    external: true
```
Let's break this up into a few pieces and talk about what's going on here. Firstly, I know that the "version" property has been deprecated but I still use it anyways, feel free to omit that in your own config files. That being said, we have our services which is just traefk in this case with the `container_name: traefik` so that it looks clean in `docker ps` and Portainer. The image I am using is `v2.11.6`.

```yaml
    ...
    ports:
      # The HTTP port
      - "80:80" # web
      - "443:443" # websecure
      # The Web UI port
      - "8081:8080"
    ...
```
These are pretty self-explanatory. I want people who still, for some reason, use http to be able to access my stuff so that's why port 80 is there. Port 443 is https and 8081 is the port I set to access the traefik web ui, with the internal port of 8080. You can use whatever port you want, assuming it's not being used for something else. In my case, I have qBittorrent on port 8080 so I'm using port 8081 here.


```yaml
    ...
    volumes:
      # Config file
      - /etc/traefik/traefik.yaml:/etc/traefik/traefik.yaml
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      # For TLS certs not to be re-generated every time...
      - /etc/traefik/certs/acme.json:/acme.json
    ...
```
These are the volume binds that I have set up. Notice that there is another `traefik.yaml` file that the container is using, that's where the rest of the set up for traefik is. Here is what that file looks like:

```yaml
global:
  checkNewVersion: true
  sendAnonymousUsage: false

# -- (Optional) Change Log Level and Format here...
#     - loglevels [DEBUG, INFO, WARNING, ERROR, CRITICAL]
#     - format [common, json, logfmt]
log:
 level: DEBUG
 format: common
 filePath: /var/log/traefik/traefik.log

# -- (Optional) Enable Accesslog and change Format here...
#     - format [common, json, logfmt]
accesslog:
  format: common
  filePath: /var/log/traefik/access.log

# -- (Optional) Enable API and Dashboard here, don't do in production
api:
  dashboard: true
  insecure: true

entryPoints:
  web:
    address: :80
    # -- (Optional) Redirect all HTTP to HTTPS
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: :443

certificatesResolvers:
  staging:
    acme:
      email: a.mehraz.cs@gmail.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web

  production:
    acme:
      email: a.mehraz.cs@gmail.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web

providers:
  docker:
    # -- (Optional) Enable this, if you want to expose all containers automatically
    exposedByDefault: false
  file:
    directory: /etc/traefik
    watch: true
```
<!-- <p align="left"> -->
\* *See [**18 July 2024 Update**]({{< ref "#18-july-2024" >}})\**


I don't feel like going through every single piece and this is also not everything you are able to set up but it's what I have. Also, yes, I know my email is there. The main parts are the entrypoints for http and https which I named `web` and `websecure`, the redirecting from http to https, and the TLS certificate generation and storage from Let's Encrypt (using an http challenge). Again, there are more options that are available to you for that like DNS Challenge with a DNS provider or TLS challenge. Learn more about that [here](https://doc.traefik.io/traefik/https/acme/).

### Updates
Any updates being made will be below. Here is a table with jumps to various dates with a short description of changes made:
> Date: Short description
>
> [18 July 2024]({{< ref "#18-july-2024" >}}): Changes to SSL

#### 18 July 2024
I have updated my traefik configuration to use my SSL certs from [Porkbun](https://porkbun.com/products/ssl) instead of the one that traefik generates. This was mostly because I wanted to know how to add my own SSL certs if in the future I did not want to use Let's Encrypt. Porkbun actually generates the cert from Let's Encrypt so I still am using them but I cared about the part where I actually tell traefik to use a particular cert and key, not really caring where/who it is sourced from. In any case, the changes I made to `traefik.yaml` include:

```yaml
certificatesResolvers:
  staging:
    acme:
      email: a.mehraz.cs@gmail.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web

  production:
    acme:
      email: a.mehraz.cs@gmail.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web
```
to &darr;
```yaml
certificatesResolvers:
  porkbun:
    acme:
      email: "a.mehraz.cs@gmail.com"
      storage: "/etc/traefik/certs/acme.json"
      dnsChallenge:
        provider: porkbun
        delayBeforeCheck: 90  # Optional, useful if DNS propagation is slow
  production:
    acme:
      email: a.mehraz.cs@gmail.com
      storage: /etc/traefik/certs/acme.json
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      httpChallenge:
        entryPoint: web

# -- Overwrites Default Certificates (above)
tls:
  stores:
    default:
      defaultCertificate:
        certFile: certs/fullchain.pem
        keyFile: certs/private.key.pem
```

I left the redundant `porkbun` field in the `certificatesResolvers` since I was using it earlier but for some reason the `acme.json` file never got propagated correctly. Instead I downloaded my "SSL Bundle" from Porkbun which includes the certificate for my domain, my public key, my private key, and the intermediate certificate. You typically only need the primary certificate and the private key. However, to ensure a complete chain of trust, you should combine your primary certificate (`domain.cert.pem`) and the intermediate certificate (`intermediate.cert.pem`) into a single file. Here’s how to prepare the files for Traefik:
```bash
cat domain.cert.pem intermediate.cert.pem > fullchain.pem && cp fullchain.pem /etc/traefik/certs/
```

\* *Note: The public.key.pem file is generally not needed for SSL/TLS configuration in Traefik.*

Now that the full certificate is where I want it, I then copied the private key into the same place as well:
```bash
cp private.key.pem /etc/traefik/certs/
```

I then made sure the permissions were correct
```bash
sudo chown 65532:65532 /etc/traefik/certs/fullchain.pem /etc/traefik/certs/private.key.pem
sudo chmod 600 /etc/traefik/certs/fullchain.pem /etc/traefik/certs/private.key.pem
```

All that was left was to edit the traefik docker compose file to include my cert
```yaml
    ...
    volumes:
      ...
      # Mount SSL certificates
      - /etc/traefik/certs/fullchain.pem:/certs/fullchain.pem
      - /etc/traefik/certs/private.key.pem:/certs/private.key.pem
    ...
```

Now traefik is able to use my SSL cert instead of one that is generated from traefik. I then told my nginx webserver to use this cert, see more in the update of [my nginx post]({{< ref "nginx#18-july-2024" >}} "nginx").