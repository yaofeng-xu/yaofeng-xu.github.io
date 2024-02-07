---
title: 基于docker部署gerrit
date: 2024-02-04 23:30:00 +0800
categories: [部署, gerrit]
tags: [CI, ldap, gerrit]
---
使用docker compose部署gerrit并配置ldap登录。
gerrit的8080端口不用映射，通过nginx反向代理来访问。
## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  gerrit:
    image: gerritcodereview/gerrit:3.8.2-ubuntu22
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    deploy:
      resources:
        limits:
          memory: 1g          # 本地部署，限制1G内存
    container_name: gerrit.docker.local
    hostname: gerrit.docker.local
    volumes:
       - /etc/localtime:/etc/localtime:ro
       - ./gerrit/etc:/var/gerrit/etc
       - ${GERRIT_DB_VOLUME}:/var/gerrit/db
       - ${GERRIT_GIT_VOLUME}:/var/gerrit/git
       - ${GERRIT_INDEX_VOLUME}:/var/gerrit/index
       - ${GERRIT_CACHE_VOLUME}:/var/gerrit/cache
    ports:
      #  - "8080:8080"        # web端口，通过nginx反向代理访问，无需映射
       - "29418:29418"        # 方便host访问
    environment:
      JAVA_OPTS: "-Xmx1g"
      CANONICAL_WEB_URL: "http://gerrit.xxx.com"
      HTTPD_LISTENURL: "http://*:8080/"
```
## 配置.env
根据实际情况配置路径
```sh
GERRIT_DB_VOLUME=/opt/volumes/gerrit/db
GERRIT_GIT_VOLUME=/opt/volumes/gerrit/git
GERRIT_INDEX_VOLUME=/opt/volumes/gerrit/index
GERRIT_CACHE_VOLUME=/opt/volumes/gerrit/cache
```
## ldap登录配置
修改容器中`/var/gerrit/etc/gerrit.config`
```sh
[auth]
	type = LDAP                             # 登录方式改为LDAP
	gitBasicAuthPolicy = HTTP               # 设置REST API认证方式
[ldap]
	server = ldap://ldap.docker.local       # ldap服务器地址
	accountBase = ou=people,dc=xxx,dc=com
	accountPattern = (&(objectClass=person)(uid=${username}))
	groupBase = ou=groups,dc=xxx,dc=com
	groupMemberPattern = (&(objectClass=group)(member=${dn}))
	username = cn=readonly,dc=xxx,dc=com    # readonly用户
	password = 123456
	accountFullName = ${givenName} ${sn}
	accountEmailAddress = mail
```
## nginx反向代理配置
在nginx配置文件目录下新建`gerrit.conf`，内容如下。
```sh
server {
    listen                      80;
    server_name                 gerrit.xxx.com;
    return                      301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               gerrit.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server_nopass.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://gerrit.docker.local:8080;
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

添加gerrit域名，前面的IP换成nginx服务器所在服务器的IP
```sh
127.0.0.1 gerrit.xxx.com
```
## 测试
HOST浏览器输入`gerrit.xxx.com`查看

![Desktop View](/static/images/202402/20240204_01.jpg){: width="1001" height="439" }

![Desktop View](/static/images/202402/20240204_02.jpg){: width="1220" height="1011" }

## 相关链接
[基于docker部署nginx](/posts/基于docker部署nginx/)  
[基于docker部署openldap](/posts/基于docker部署openldap/)  