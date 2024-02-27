---
title: Gerrit和Jenkins集成
date: 2024-02-27 22:31:00 +0800
categories: [CI, jenkins]
tags: [CI, jenkins, gerrit]
---
## Gerrit配置
### 登录并配置gerrit并配置ssh key  
可参考  
[Gerrit用户SSH_KEY配置](/posts/Gerrit用户SSH_KEY配置/) 
### 将jenkins用户配置为service user
使用 gerrit的admin用户登录并操作  
![Desktop View](/static/images/202402/20240227_01.jpg)
## Jenkins配置
### 安装Gerrit Trigger插件
![Desktop View](/static/images/202402/20240227_02.jpg)
### 配置Gerrit Trigger
插件安装完成后系统管理拉到最下面可找到  
![Desktop View](/static/images/202402/20240227_03.jpg)

![Desktop View](/static/images/202402/20240227_04.jpg)

![Desktop View](/static/images/202402/20240227_05.jpg)

![Desktop View](/static/images/202402/20240227_06.jpg)  
ssh key 私钥与上面gerrit中的公钥要一致  
点击Test Connect测试连接情况  
全部填完拉到最后点击保存

![Desktop View](/static/images/202402/20240227_07.jpg)  
如图小圆点能够点蓝表示可以连接成功

## 相关链接
[基于docker部署gerrit](/posts/基于docker部署gerrit/)  
[基于docker部署jenkins](/posts/基于docker部署jenkins/)  
[Gerrit用户SSH_KEY配置](/posts/Gerrit用户SSH_KEY配置/)  