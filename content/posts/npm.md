+++
draft = false
date = "2024-09-07"
title = "How To Setup Nginx Proxy Manager Using Portainer (Docker) "
description = "How to use nginx proxy manager to reverse proxy."
slug = "npm"
authors = ["Jukelyn"]
categories = ["Nginx", "Reverse Proxy", "Website"]
+++

[Jump to Updates]({{< ref "#updates" >}})
### What is Nginx Proxy Manager?
Nginx Proxy Manager (NPM) is a web-based, easy-to-use tool that simplifies the management of Nginx reverse proxies. It allows users to manage their reverse proxies, SSL certificates, and forwarding rules with a graphical user interface (GUI), making it much easier for beginners or non-experts to set up and manage web servers. Here's an overview of what Nginx Proxy Manager offers:

1. Reverse Proxy Setup
    - Nginx Proxy Manager helps you configure Nginx as a reverse proxy to forward web traffic from one domain or port to another service running behind it. This is especially useful for routing requests to different services (like websites, applications, or APIs) hosted on the same server.

2. SSL Management
    - It makes obtaining and managing SSL certificates (e.g., from Let's Encrypt) straightforward. You can easily enable HTTPS for your domains, ensuring secure communication between clients and your services.

3. User-Friendly Interface
    - The web-based GUI simplifies many tasks that would normally require manual editing of Nginx configuration files. This is particularly useful for people who aren't comfortable working directly with terminal commands or config files.
4. Access Control
    - NPM allows you to set up basic authentication to protect your web services behind login credentials, adding a layer of security to your applications.
5. Custom Domain Support
    - You can easily route traffic based on domain names. For example, you can have `app1.yourdomain.com` go to one server and `app2.yourdomain.com` go to another.
6. Multiple Protocols
    - Besides HTTP and HTTPS, Nginx Proxy Manager can also handle WebSocket connections and manage TCP/UDP proxying.
7. Logs and Monitoring
    - It provides access to logs, making it easier to monitor traffic, troubleshoot issues, and see where requests are being routed.

### Typical Use Cases
1. Self-hosted Applications: If you're hosting multiple services (e.g., web apps, Docker containers, APIs) on the same server, Nginx Proxy Manager helps to route traffic to the correct service based on the domain or subdomain.

2. SSL/TLS Management: For anyone running public-facing websites, Nginx Proxy Manager automates SSL certificate generation and renewal via Let's Encrypt.

3. Securing Web Services: You can quickly configure secure reverse proxies for services running behind firewalls, like private applications or administrative panels.

I am going to use NPM to replace my current reverse proxy, Traefik. Doing this is pretty simple, first we need to deploy NPM in a docker container and let it have access to the same Docker networks that I have.

```yaml
services:
  npm:
    container_name: npm
    image: jc21/nginx-proxy-manager:latest
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
      - 81:81 # Admin Web Port
    environment:
      DISABLE_IPV6: true
    volumes:
      - ./data/npm/data:/data
      - ./data/letsencrypt:/etc/letsencrypt
    networks:
      - default
      - vault
      - sites
      - kuma_net
      - immich
    healthcheck:
      test: ["CMD", "/usr/bin/check-health"]
      interval: 10s
      timeout: 3s
...
```

This is pretty simple to understand but the key parts are the ports where we have 80 for HTTP, 443 for HTTPS, and 81 for the admin web UI. The networks that I have listed are the networks that I have various services running on that I have to have subdomains pointing to those services. Once I deploy this, I can navigate to my local ip on that port to access the admin web UI, so `http://192.168.1.87:81`.

### Initial Setup
Once you navigate to the admin web UI for the first time you will be met with the login page, the default credentials are:
```
email:    admin@example.com
Password: changeme
```

Immediately after logging in with this default user you will be asked to modify your details and change your password.

### Setting up services
Setting up services in NPM is really simple. I will showing you how to setup a subdomain for NPM itself on my domain `jukelyn.com`.

Navigate to "Hosts" on the top bar and click on "Add Proxy Host". The domain name will be `npm.jukelyn.com` with the scheme `http`, with the `Forward Hostname / IP` being `npm`. This hostname is the name of the docker container that NPM is running on, I named mine "npm", thus I will enter "npm", change this as needed. The port will be set to port 81 since that is the port of the NPM admin web UI.

By default, the "Access List" is set to "Publicly Accessible". This is fine for now as we will address this next. For the SSL tab, I will enter "Request a new SSL Certificate" and then enable Force SSL, HTTP/2 Support, and HSTS.

Now go to the "Access Lists" tab in NPM, create a new access list and name it something meaningful, mine is going to be named "Internal". In the "Access" tab, I will then set it up so that it has `allow` for all my local traffic which, for me, is `192.168.1.0/24`. Save and exit.

Now change the NPM proxy host `npm.jukelyn.com` to use this Access List instead of the default public one. I am now able to enter `npm.jukelyn.com` to go to the NPM admin web UI.

You can set up other services similarly using the hostname, if on the same docker network (as I have it set up), or IP.

### Updates
Any updates being made will be below. Here is a table with jumps to various dates with a short description of changes made:

> Date: Short description
>
> TBD: TBA
