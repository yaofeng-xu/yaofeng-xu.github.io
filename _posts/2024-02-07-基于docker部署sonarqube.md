---
title: 基于docker部署SonarQube
date: 2024-02-07 00:32:00 +0800
categories: [CI, 部署]
tags: [CI, ldap, SonarQube]
---
使用docker compose部署SonarQube并配置ldap登录。
SonarQube的9000端口不用映射，通过nginx反向代理来访问。
## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  sonar:
    image: sonarqube:10-community
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    deploy:
      resources:
        limits:
          memory: 4g          # sonar内置了Elasticsearch，内存需求比较大
    container_name: sonar.docker.local
    hostname: sonar.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${SONAR_CONF_VOLUME}:/opt/sonarqube/conf
      - ${SONAR_EXT_VOLUME}:/opt/sonarqube/extensions
      - ${SONAR_LOGS_VOLUME}:/opt/sonarqube/logs
      - ${SONAR_DATA_VOLUME}:/opt/sonarqube/data
    # ports:
    #   - "9000:9000"         # web端口，通过nginx反向代理访问，无需映射
    command:
      # 设置服务代理路径
      - -Dsonar.web.context=/
      # 此设置用于集成gitlab时，回调地址设置
      - -Dsonar.core.serverBaseURL=http://sonar.docker.local
```
## 配置.env
根据实际情况配置路径
```sh
SONAR_CONF_VOLUME=/opt/volumes/sonarqube/conf
SONAR_EXT_VOLUME=/opt/volumes/sonarqube/extensions
SONAR_LOGS_VOLUME=/opt/volumes/sonarqube/logs
SONAR_DATA_VOLUME=/opt/volumes/sonarqube/data
```
## ldap登录配置
修改容器中`/opt/sonarqube/sonar.properties`
```sh
# LDAP CONFIGURATION
# Enable the LDAP feature
sonar.security.realm=LDAP
# URL of the LDAP server. Note that if you are using ldaps, then you should install the server certificate into the Java truststore.
ldap.url=ldap://ldap.docker.local:389
# Bind DN is the username of an LDAP user to connect (or bind) with. Leave this blank for anonymous access to the LDAP directory (optional)
ldap.bindDn=cn=readonly,dc=xxx,dc=com
# Bind Password is the password of the user to connect with. Leave this blank for anonymous access to the LDAP directory (optional)
ldap.bindPassword=123456
# Distinguished Name (DN) of the root node in LDAP from which to search for users (mandatory)
ldap.user.baseDn=ou=people,dc=xxx,dc=com
# LDAP user request. (default: (&(objectClass=inetOrgPerson)(uid={login})) )
ldap.user.request=(&(objectClass=person)(uid={login}))
# Distinguished Name (DN) of the root node in LDAP from which to search for groups. (optional, default: empty)
ldap.group.baseDn=ou=groups,dc=xxx,dc=com
# LDAP group request (default: (&(objectClass=groupOfUniqueNames)(uniqueMember={dn})) )
ldap.group.request=(&(objectClass=group)(member=${dn}))
```
## nginx反向代理配置
在nginx配置文件目录下新建`sonar.conf`，内容如下。
```sh
server {
    listen                    80;
    server_name               sonar.xxx.com;
    return                    301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               sonar.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://sonar.docker.local:9000;
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

添加sonar域名，前面的IP换成nginx服务器所在服务器的IP
```sh
127.0.0.1 sonar.xxx.com
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
HOST浏览器输入`sonar.xxx.com`查看  
首次登录  
用户名: admin  
密  码: admin  
![Desktop View](/static/images/202402/20240207_01.jpg)  
使用H2数据库会出现如下提示  
![Desktop View](/static/images/202402/20240207_02.jpg)  

## 相关链接
[基于docker部署nginx](/posts/基于docker部署nginx/)  
[基于docker部署openldap](/posts/基于docker部署openldap/)  
[基于docker部署postgresql和adminer](/posts/基于docker部署postgresql和adminer/)  