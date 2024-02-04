---
title: 基于docker部署openldap
date: 2024-02-03 20:59:00 +0800
categories: [CI, 部署]
tags: [CI, ldap]
---
使用docker compose部署openldap和openldap web客户端
## 配置docker-compose.yml {#config-docker-compose}
部署`osixia/openldap:1.5.0`，openldap服务
部署`ldapaccountmanager/lam:8.3`，openldap web客户端
```yaml
version: "3"
name: common

services:
  ldap:
    image: osixia/openldap:1.5.0
    restart: unless-stopped
    logging:
      driver: none                      # 本地部署，不启用日志
    container_name: ldap.docker.local
    hostname: ldap.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${LDAP_DB_VOLUME}:/var/lib/ldap
      - ${LDAP_CFG_VOLUME}:/etc/ldap/slapd.d
    environment:
      LDAP_ORGANISATION: xxx
      LDAP_DOMAIN: xxx.com
      LDAP_ADMIN_PASSWORD: 123456
      LDAP_CONFIG_PASSWORD: 123456
      LDAP_READONLY_USER: true
      LDAP_READONLY_USER_USERNAME: readonly
      LDAP_READONLY_USER_PASSWORD: 123456

  lam:
    image: ldapaccountmanager/lam:8.3
    restart: unless-stopped
    logging:
      driver: none                      # 本地部署，不启用日志
    container_name: lam.docker.local
    hostname: lam.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
    environment:
      LAM_LANG: zh_CN
      LAM_PASSWORD: 123456
      LDAP_SERVER: ldap://ldap.docker.local:389
      LDAP_USER: cn=admin,dc=xxx,dc=com
      LDAP_DOMAIN: xxx.com
      LDAP_BASE_DN: dc=xxx,dc=com
      LDAP_USERS_DN: ou=people,dc=xxx,dc=com
      LDAP_GROUPS_DN: ou=groups,dc=xxx,dc=com
```
启动可通过一下命令测试openldap是否成功
```sh
[root@localhost keys]# docker exec ldap.docker.local ldapsearch -x -H ldap://localhost -b dc=xxx,dc=com -D "cn=admin,dc=xxx,dc=com" -w 123456
...
```
## 配置.env
根据实际情况配置路径
```sh
LDAP_DB_VOLUME=/opt/volume/ldap/db
LDAP_CFG_VOLUME=/opt/volume/ldap/cfg
```
## nginx反向代理配置
在nginx配置文件目录下新建`lam.conf`，内容如下。
```sh
server {
    listen                      80;
    server_name                 lam.xxx.com;
    return                      301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               lam.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server_nopass.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://lam.docker.local;
        proxy_pass            $target;
        resolver              127.0.0.11;   # 设置容器dns，否则会找不到Gerrit服务器
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
```sh
127.0.0.1 localhost
127.0.0.1 lam.xxx.com       # 添加lam域名，前面的IP换成nginx服务器所在服务器的IP
```

## 测试
HOST浏览器输入`lam.xxx.com`查看

![Desktop View](/static/images/202402/20240203_01.jpg){: width="800" height="579" }

![Desktop View](/static/images/202402/20240203_02.jpg){: width="870" height="675" }
