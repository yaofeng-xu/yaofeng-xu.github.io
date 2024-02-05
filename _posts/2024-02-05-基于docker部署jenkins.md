---
title: 基于docker部署jenkins
date: 2024-02-05 22:30:00 +0800
categories: [CI, 部署]
tags: [CI, ldap, jenkins]
---
使用docker compose部署jenkins并配置ldap登录。
jenkins的8080端口不用映射，通过nginx反向代理来访问。
## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  jenkins:
    image: jenkins/jenkins:2.426.1-slim-jdk17
    restart: unless-stopped
    logging:
      driver: none            # 本地部署，不启用日志
    privileged: true
    deploy:
      resources:
        limits:
          memory: 1g          # 本地部署，限制1G内存
    container_name: jenkins.docker.local
    hostname: jenkins.docker.local
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ${JENKINS_HOME_VOLUME}:/var/jenkins_home
    environment:
      JAVA_OPTS: "-Xmx1g"
    # ports:
    #   - "8080:8080"         # web端口，通过nginx反向代理访问，无需映射
    #   - "50000:50000"       # agent端口，docker bridge内部访问，无需映射
```
## 配置.env
根据实际情况配置路径
```sh
JENKINS_HOME_VOLUME=/opt/volumes/jenkins
```
## nginx反向代理配置
在nginx配置文件目录下新建`gerrit.conf`，内容如下。
```sh
server {
    listen                    80;
    server_name               jenkins.xxx.com;
    return                    301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               jenkins.xxx.com;
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;
    client_max_body_size      20M;

    location / {
        set $target           http://jenkins.docker.local:8080;
        proxy_pass            $target;
        resolver              127.0.0.11;   # 设置容器dns，否则会找服务器
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

添加jenkins域名，前面的IP换成nginx服务器所在服务器的IP
```sh
127.0.0.1 jenkins.xxx.com
```
## ldap登录配置
首次登录admin用户密码路径  
`/var/jenkins_home/secrets/initialAdminPassword`中。  
安装默认插件，在`系统管理 > 全局安全配置`中配置LDAP。  
配置如下：  
![Desktop View](/static/images/202402/20240205_01.jpg){: width="953" height="2439" }

## 相关链接
[基于docker部署nginx](/posts/基于docker部署nginx/)  
[基于docker部署openldap](/posts/基于docker部署openldap/)  