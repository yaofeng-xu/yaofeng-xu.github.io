---
title: 基于docker部署postgresql
date: 2024-02-07 01:00:00 +0800
categories: [CI, 部署]
tags: [CI, database, postgresql]
---
使用docker compose部署postgresql和adminer。
adminer的9000端口不用映射，通过nginx反向代理来访问。
## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  pgsql:
    image: postgres:12.17-alpine
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    container_name: pgsql.docker.local
    hostname: pgsql.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
    # ports:
    #   - "5432:5432"         # 数据库访问端口，docker内部网络使用，无需映射
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: 123456

  adminer:
    image: amd64/adminer:4.8.1
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    container_name: adminer.docker.local
    hostname: adminer.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
    # ports:
    #   - "8080:8080"         # web端口，通过nginx反向代理访问，无需映射
```
## nginx反向代理配置
在nginx配置文件目录下新建`adminer.conf`，内容如下。
```sh
server {
    listen                    80;
    server_name               adminer.xxx.com;
    return                    301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               adminer.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://adminer.docker.local:8080;
        proxy_pass            $target;
        resolver              127.0.0.11;
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

添加adminer域名，前面的IP换成nginx服务器所在服务器的IP
```sh
127.0.0.1 adminer.xxx.com
```
## 数据库配置
SonarQube默认使用H2数据库，需要用外部数据库参考如下设置
```sh
# User credentials.
# Permissions to create tables, indices and triggers must be granted to JDBC user.
# The schema must be created first.
sonar.jdbc.username=root        # 数据库用户名
sonar.jdbc.password=123456      # 数据库密码

#----- PostgreSQL 11 or greater
# By default the schema named "public" is used. It can be overridden with the parameter "currentSchema".
sonar.jdbc.url=jdbc:postgresql://pgsql.docker.local/sonar # 数据库链接，sonar是数据库名称
```
## 测试
HOST浏览器输入`adminer.xxx.com`进行数据库操作  
![Desktop View](/static/images/202402/20240207_03.jpg)  
创建数据库  
![Desktop View](/static/images/202402/20240207_04.jpg)  

## 相关链接
[基于docker部署nginx](/posts/基于docker部署nginx/)  
[基于docker部署sonarqube](/posts/基于docker部署sonarqube/)  