---
title: VirtualBox中Ubuntu自动挂载共享文件夹
date: 2024-02-24 22:31:00 +0800
categories: [Linux, mount]
tags: [Linux, mount, VirtualBox]
---

## VirtualBox中共享Windows目录
![Desktop View](/static/images/202402/20240224_04.jpg) 
## 手动挂载测试
启动Ubuntu虚拟机  

```sh
$ sudo mkdir -p /mnt/d
# 'D_DRIVE' 是上面填写的共享文件夹名称
$ sudo mount -t vboxsf D_DRIVE /mnt/d
# 此时查看 /mnt/d 目录就会看到你共享的Windows文件夹的内容
```
## 开机自动挂载
编辑`/etc/fstab`，在末尾添加
```sh
D_DRIVE /mnt/d vboxsf defaults 0 0
```
重启，`D_DRIVE`自动挂载到`/mnt/d`下