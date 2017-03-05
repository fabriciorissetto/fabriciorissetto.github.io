---
layout: post
title: "Rodando NGINX com Docker no Windows"
modified:
categories: blog
excerpt: ""
tags: [Docker]
image:
  feature:
date: 2016-09-01T15:39:55-04:01
---

Intro sobre pq docker é cool

```shell
docker run -p 80:80 -d nginx
```

```shell
docker ps
```

$(docker-machine ip default)
Mostrar rodando na pota 80

```shell
docker stop
```

http://stackoverflow.com/questions/33312662/docker-toolbox-mount-file-on-windows
 docker run -d --name parking-app -v /C/Projects/Git/parking-app/nginx:/etc/nginx/conf.d/ -v /C/Projects/Git/parking-app/dist:/usr/share/nginx/html --add-host="api:10.99.2.147" -p 8083:80 nginx
 docker run -d --name parking-app-neto -v $(pwd)/nginx/nginx.conf.off:/etc/nginx/nginx.conf -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf -v $(pwd)/dist://usr/share/nginx/html --add-host="api:10.99.2.147" -p 8086:80 nginx
http://192.168.99.100:8083/

 
 # conectar no container (direto no container docker que tá dentro da VM linux)
 docker exec -it parking-app bash 

 
 docker build -t some-content-nginx .
 
docker run -d --name some-nginx -p 8083:80 fabricio-nginx
