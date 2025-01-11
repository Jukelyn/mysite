+++
draft = false
date = "2024-06-21"
title = "How To Setup Nginx As A Webserver Using Portainer (Docker)"
description = "How to use nginx webserver to host static sites."
slug = "nginx"
authors = ["Jukelyn"]
categories = ["nginx","webserver","Website"]
+++

[Jump to Updates]({{< ref "#updates" >}})
### What is nginx?

Nginx (pronounced "engine X") is an open-source web server software known for its high performance, stability, and low resource usage. It serves static content, acts as a reverse proxy and load balancer, and can cache HTTP responses to improve speed and efficiency. Its simple configuration and ability to handle many simultaneous connections make it ideal for high-traffic websites. Nginx is widely used due to its flexibility and robust feature set, with both open-source and enterprise versions available. In this post I will only be using it as a web server to host a static site (the one you are on right now).

### Installing and setting up nginx

There are multiple options when it comes to installing and using nginx but I prefer to have it installed within a docker container. As you may already know from my [traefik post]({{< ref "traefik" >}} "traefik"), I prefer using a yaml docker compose file for my containers.

This is what my full nginx `docker-compose.yml` looks like:
```yaml
version: '3'

services:
  nginx:
    container_name: nginx
    image: nginx:latest
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nginx.rule=Host(`jukelyn.com`)"
      - "traefik.http.routers.nginx.entrypoints=web,websecure"
      - "traefik.http.routers.nginx.tls=true"
      - "traefik.http.routers.nginx.tls.certresolver=production"
    volumes:
      - /home/juke/sites/mysite/public:/usr/share/nginx/html
#      - /home/juke/nginx/nginx.conf:/etc/nginx/nginx.conf
    restart: always
    networks:
      - nginx

networks:
  nginx:
    external: true
```
Let's break this up into a few pieces and talk about what's going on here. Firstly, I know that the "version" property has been deprecated but I still use it anyways, feel free to omit that in your own config files. That being said, we have our services which is just nginx in this case with the `container_name: nginx` so that it looks clean in `docker ps` and Portainer. The image I am using is one of the versions that is most recent as of writing this but I was using the `latest` tag (instead of `stable-perl`) in the past and it worked just fine as well.

Now for this part...
```yaml
    ...
    labels:
      - traefik.enable=true
      - traefik.http.routers.nginx.rule=Host(`jukelyn.com`)
      - traefik.http.routers.nginx.entrypoints=web,websecure
      - traefik.http.routers.nginx.tls=true
      - traefik.http.routers.nginx.tls.certresolver=production
    ...
    networks:
      - nginx
```
The above information is there for this webserver to work with my reverse proxy, [traefik]({{< ref "traefik" >}} "traefik"). Without getting into it much this basically lets the container know to be on the same network as traefik and the labels are there to let the container know to use enable traefik through `jukelyn.com` on entrypoints `web` and `websecure` which are ports 80 and 443 (configured and named in traefik), lastly use tls for encryption (http vs https).

```yaml
    ...
    volumes:
      - /home/juke/sites/mysite/public:/usr/share/nginx/html
    restart: always
    ...
```
Lastly, the remaining parts are just for the container restart policy to always and the volume is the path to my site's html files and it's mapped to the directory that nginx looks for files to show, which is in the container. Overall, it's pretty simple and easy to set up.

### Updates
Any updates being made will be below. Here is a table with jumps to various dates with a short description of changes made:

> Date: Short description
>
> [18 July 2024]({{< ref "#18-july-2024" >}}): Changes to SSL

#### 18 July 2024
I have updated my [traefik configuration]({{< ref "traefik#18-july-2024" >}} "traefik") to use my SSL certs and so the `traefik.http.routers.nginx.tls.certresolver=production` label is actually overwritten with my SSL cert. 