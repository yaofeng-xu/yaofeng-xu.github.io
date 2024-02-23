---
title: GERRIT用户SSH KEY配置
date: 2024-02-23 23:12:00 +0800
categories: [部署, gerrit]
tags: [git, gerrit, ssh]
---

## 创建SSH KEY
```sh
$ ssh-keygen -t rsa -C "york@xxx.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/york/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/york/.ssh/id_rsa
Your public key has been saved in /home/york/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:+O0ApJkaiXNPJ/wKvjjGT/Qet2kZ3Ml/gTXc81wpMpA york@xxx.com
The key's randomart image is:
+---[RSA 3072]----+
|          .      |
|         E       |
|      .   . . . .|
| . o = .   o = +.|
|o +.B = S . = o.+|
| o.=.+ = = . .  o|
|. o..o..= o   .  |
|.=....oooo . .   |
|o.+o...o  . .    |
+----[SHA256]-----+
```
## 添加SSH KEY到 gerrit
```sh
$ cat ~/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC+WRRsnK+179aGlx32svQ1DmTgi7BFk3LuiiblFHkOchszRgunOUdB+SOXVEbAgq9WBKOiRTxOxI+JtkzA4spcZN86orFNKO4TLSwV7zYBoCAAkJeMSrvy4vRhySkkfwwJ2o9Kn6uNGTWTQu8QH4K2qoRzqXVufKD7disHAzpd/Be/0q+Iwkj9GWrr70xZtAIR14iThUhhkiLwVtGQ4HERNqkyZs02qTtFg5nFNGh2ljoA5w7sFLsEV4z9e2pTJ+skrXq7XC01ttFWAz6+DcxhmmTotJUTN4fgihzqyO86nSLNNtHpv3T//GvkUhtbMX1Eoi70eSucvmGQd7beinhuN3I19l+ZWcs3JpNUz9sovyPws22rS5huLABnrzBKv1mGCwaHX3DMMqsWrHoQqhBHdMl/Z2iVMoJb22k9hN4+UwGNA5I3u/YsHnjguHyANQqHTNx6a7jFhxuA0Lnun+OUJLBqDtZuOkZvf8bCyeT/7XHzCPjl/L0DXwRU/GEh/d8= york@xxx.com
```
将上面的内容添加到Gerrit如图  
![Desktop View](/static/images/202402/20240224_01.jpg) 

## gerrit用户创建
### 使用LDAP方式登录Gerrit
参考前面的文章  
[基于docker部署gerrit](/posts/基于docker部署gerrit/)  
[基于docker部署openldap](/posts/基于docker部署openldap/)  
### 开发环境，自用
自己随便用用也可以将`/var/gerrit/etc/gerrit.config`做如下配置  
```sh
# DEVELOPMENT_BECOME_ANY_ACCOUNT
# DO NOT USE. Only for use in a development environment.
[auth]
	type = DEVELOPMENT_BECOME_ANY_ACCOUNT
```
![Desktop View](/static/images/202402/20240224_02.jpg) 
### 用htpasswd创建用户
参考下面链接
https://www.cnblogs.com/juandx/p/5195543.html

## 相关链接
[基于docker部署gerrit](/posts/基于docker部署gerrit/)  
[基于docker部署openldap](/posts/基于docker部署openldap/) 