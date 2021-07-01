---
layout: post
title: "Ansible 常用模块说明"
subtitle: '运维日常脚本使用Ansible常用模块的一些记录'
author: "chaoxiaodi"
header-style: text
tags:
  - ansible
  - shell
---

### 前言
在日常运维工作中在对一些任务进行自动化操作时

会使用到ansible的一些功能

已经写过一篇通过python调用ansible的文章

文章链接：[运维管理平台-任务处理](https://blog.chaoxiaodi.tech/2021/03/15/python-ansible-api%E8%B0%83%E7%94%A8%E5%AE%9E%E7%8E%B0/)

ansible 是python写的一个便于运维自动化的开源工具



### 常用模块

#### ping

此模块可以说是ansible界的 'hello world'

每个开始使用ansible的人开始测试的第一个模块基本都是ping

    eg: ansible test1 -m ping
    说明：
    使用ansible需要一些前期的准备工作
    比如 SSH免密登录，ansible一些文件配置 hosts-groups-ip 这些的对应关系
    在日常使用中可以通过指定机器名、机器分组、ip 对远程机器进行操作
    
    这个例子的代表 对 test1 主机进行ping操作

#### command

看这个模块名称就知道是用来执行一些命令的模块

这个模块可以执行一些简单的命令

但无法支持一些特殊符号，如：无法支持"<"，">"，"|"，";"，"&"等符号

#### shell

shell 模块可以看作是command模块的升级进阶版

可以支持上述无法支持的特殊符号

也是本人日常使用中用的最多的模块

#### file

file模块用来创建文件、目录、链接文件

除了创建还可以修改文件的一些属性

如 属组，权限，属主等

这个模块我个人用的还是比较少的

#### copy

 copy模块用来复制文件至目标主机
 
 看这个模块名称也是一眼就知道作用
 
 理解为 SCP 命令
 
 这个模块有一个比较好用的功能就是 backup 备份功能
 
 如果设置yes，远程主机如果存在同名文件就会自动备份一个
 
#### service

service模块用来管理centos上的服务的启动、关闭、重启和重载

可以对应 systemctl 命令

对远程主机进行服务的启动(started)、停止(stoped)、重启(restarted)、开机自启动等
#### user

用来管理用户，用的较少

#### group

用来管理用户组，用的较少

#### unarchive

unarchive模块用来解压文件

这个一般较少使用，可以作为了解

### 后记

本文内容不是很多

主要为记录提醒用

具体模块需要根据日常使用的过程进行测试后使用

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
