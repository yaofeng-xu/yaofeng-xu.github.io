---
title: 基于docker部署gerrit
date: 2024-02-07 20:57:00 +0800
categories: [部署, nexus]
tags: [CI, ldap, nexus]
---
使用docker compose部署nexus并配置ldap登录。
nexus的8081端口不用映射，通过nginx反向代理来访问。
## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  nexus:
    image: sonatype/nexus3:3.64.0
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    container_name: nexus.docker.local
    hostname: nexus.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${NEXUS_DATA_VOLUME}:/nexus-data
    # ports:
    #  - "8081:8081"          # web端口，通过nginx反向代理访问，无需映射
```
## 配置.env
根据实际情况配置路径
```sh
NEXUS_DATA_VOLUME=/opt/volumes/nexus/data
```
## nginx反向代理配置
在nginx配置文件目录下新建`gerrit.conf`，内容如下。
```sh
server {
    listen                    80;
    server_name               nexus.xxx.com;
    return                    301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               nexus.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://nexus.docker.local:8081;
        proxy_pass            $target;
        resolver              127.0.0.11;   # 设置容器dns，否则会找不到服务器
        proxy_redirect        off; 
        proxy_set_header      Host $host; 
        proxy_set_header      X-Real-IP $remote_addr; 
        proxy_set_header      X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header      X-Forwarded-Proto $scheme;
    }
}
```
## 本地HOST添加网址
我的HOST是WIN11，`hosts`路径如下。
> C:\Windows\System32\drivers\etc\hosts

添加nexus域名，前面的IP换成nginx服务器所在服务器的IP
```sh
127.0.0.1 nexus.xxx.com
```
## ldap登录配置
HOST浏览器输入`nexus.xxx.com`访问
默认用户名莫玛
用户名: admin  
密  码: admin  

![Desktop View](/static/images/202402/20240207_05.jpg)  

![Desktop View](/static/images/202402/20240207_06.jpg)

![Desktop View](/static/images/202402/20240207_07.jpg)

## 相关链接
[基于docker部署nginx](/posts/基于docker部署nginx/)  
[基于docker部署openldap](/posts/基于docker部署openldap/)  