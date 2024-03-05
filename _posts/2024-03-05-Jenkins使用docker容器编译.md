---
title: Jenkins使用docker容器编译
date: 2024-03-05 23:00:00 +0800
categories: [CI, jenkins]
tags: [CI, jenkins, docker]
---
## 插件Docker安装
在插件管理->Available plugins中搜索docker,并安装插件
![Desktop View](/static/images/202403/20240305_03.jpg)  
## 配置Clouds
1. 点击Clouds  
![Desktop View](/static/images/202403/20240305_02.jpg)  
2. 点击新建  
![Desktop View](/static/images/202403/20240305_04.jpg)  
3. 创建Cloud  
- 填写cloud名称  
- 类型选择docker  
- 点击创建  
![Desktop View](/static/images/202403/20240303_01.jpg)  
4. 详细配置  
已贴出所有配置供参考  
![Desktop View](/static/images/202403/20240305_01.jpg)  
## 测试
1. 创建流水线任务  
![Desktop View](/static/images/202403/20240305_05.jpg)  
2. 使用pipline demo  
![Desktop View](/static/images/202403/20240305_06.jpg)  
保存
3. 构建  
点击立即构建按钮  
4. 查看结果  
![Desktop View](/static/images/202403/20240305_06.jpg)  
## 执行问题  
1. 如果任务等待好久都没执行  
可能是容器启动出问题了。去创建的clouds status页面可以看到错误信息，依据提示解决问题。
2. 没在配置的Cloud上执行  
将master节点的 用法改成`只允许允许绑定到这台机器的Job`试试。

## 相关链接
[基于docker部署gerrit](/posts/基于docker部署gerrit/)  
[基于docker部署jenkins](/posts/基于docker部署jenkins/)  
[Gerrit用户SSH_KEY配置](/posts/Gerrit用户SSH_KEY配置/)  