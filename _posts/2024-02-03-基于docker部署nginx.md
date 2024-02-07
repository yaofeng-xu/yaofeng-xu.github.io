---
title: 基于docker部署nginx
date: 2024-02-03 20:59:00 +0800
categories: [部署, nginx]
tags: [CI, nginx, proxy]
---
使用docker compose部署nginx，反向代理本地的CI相关服务。

## 目录结构
```sh
.
├── .env
├── docker-compose.yml
└── nginx
    ├── conf.d
    │   ├── default.conf
    │   └── demo.conf
    └── keys
        ├── server.crt
        ├── server.csr
        ├── server.key
        └── server_nopass.key
```

## 配置docker-compose.yml
```yaml
version: "3"
name: common

services:
  nginx:
    image: nginx:1.22.0-alpine
    restart: always
    logging:
      driver: none                      # 本地部署，不启用日志
    container_name: proxy.docker.local
    hostname: proxy.docker.local
    ports:
      - "80:80"
      - "443:443"                       # 不使用https代理可去掉本行
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/keys:/etc/nginx/keys    # 不使用https代理可去掉本行
```

## default.conf内容
```sh
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	index index.html index.htm index.nginx-debian.html;

	server_name _;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

## demo.conf内容
代理站点的模板
```sh
server {
    listen                    80;
    server_name               demo.xxx.com;             # 替换网址
    location / {
        set $target           http://demo.docker.local; # 替换本地服务地址
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
> 本地http反向代理到此已经完成，需要`https`代理的继续往下看。

---

## 自建SSL证书
本地测试使用，这边使用openssl生成测试证书
### 1.生成一个RSA密钥
```sh
[root@localhost keys]#  openssl genrsa -des3 -out server.key 1024
Generating RSA private key, 1024 bit long modulus
.......++++++
...++++++
e is 65537 (0x10001)
Enter pass phrase for server.key:               #输入密码，自定义，不少于4个字符
Verifying - Enter pass phrase for server.key:   #确认密码
```
### 2.生成一个证书请求
```sh
[root@localhost keys]# openssl req -new -key server.key -out server.csr
Enter pass phrase for server.key:               #输入刚刚创建的秘密码
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN                        #国家名称
State or Province Name (full name) []:ShangHai              #省
Locality Name (eg, city) [Default City]:ShangHai            #市
Organization Name (eg, company) [Default Company Ltd]:ACBC  #公司
Organizational Unit Name (eg, section) []:Tech              #部门
#注意，此处应当填写你要部署的域名，如果是单个则直接添加即可，如果不确定，使用*，表示可以对所有xxx.com的子域名做认证
Common Name (eg, your name or your server's hostname) []:*.xxx.com
Email Address []:admin@xxx.com                          #以域名结尾即可

Please enter the following 'extra' attributes
to be sent with your certificate request 
A challenge password []:                                #是否设置密码，可以不写直接回车  
An optional company name []:                            #其他公司名称 可不写
```
### 3.创建免密RSA证书
需要创建免密的RSA证书，否则每次reload、restart都需要输入密码
```sh
[root@localhost keys]# openssl rsa -in server.key -out server_nopass.key
Enter pass phrase for server.key:                       #之前RSA秘钥创建时的密码
writing RSA key
```
### 4.签发证书
测试环境自己签发，实际应该将自己生成的csr文件提交给SSL认证机构认证
```sh
[root@localhost keys]# openssl x509 -req -days 3650 -in server.csr  -signkey server.key -out server.crt    
Signature ok
subject=/C=CN/ST=ShangHai/L=ShangHai/O=ACBC/OU=Tech/CN=*.xxx.com/emailAddress=admin@xxx.com
Getting Private key
Enter pass phrase for server.key:                       #RSA创建时的密码
```

## https反向代理配置
```sh
server {
    listen                    80;
    server_name               demo.xxx.com;             # 替换网址
    return                    301 https://$server_name$request_uri;
}

server {
    listen                    443 ssl;
    server_name               demo.xxx.com;             # 替换网址
    ssl_certificate           /etc/nginx/keys/server.crt;
    ssl_certificate_key       /etc/nginx/keys/server_nopass.key;
    ssl_session_cache         shared:SSL:1m;
    ssl_session_timeout       5m;
    ssl_protocols             SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        set $target           http://demo.docker.local; # 替换本地服务地址
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