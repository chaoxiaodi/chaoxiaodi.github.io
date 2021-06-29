---
layout: post
title: "Python 编译安装"
subtitle: '此文记录下python3的编译安装过程'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - 编译
---

### 前言

在工作日常使用python的时候，我个人的习惯是使用python3

但现在大部分流行的linux发行版自带的python仍然是python2

由于系统会依赖一些python2的功能

所以不能直接把python2卸载然后安装python3

需要让python2与python3共存

### 安装
####下载安装包

python安装包所有版本下载地址

https://www.python.org/downloads/release

本次以下载3.9.6版本为例

下载地址：https://www.python.org/ftp/python/3.9.6/Python-3.9.6.tar.xz

在linux使用wget进行下载

wget https://www.python.org/ftp/python/3.9.6/Python-3.9.6.tar.xz

#### 解压&编译&安装

    解压：
    tar -xvJf Python-3.9.5.tar.xz
    进入目录：
    cd Python-3.9.6
    预编译
    ./configure prefix=/usr/local/python3
    编译安装 可以通过指定 cpu数量加快速度
    make -j4 && make install
    
    tips:
    此过程如果出现报错，一般情况是依赖问题
    可以根据具体报错安装具体依赖包
    然后重复此步骤

#### 添加环境变量

    想要使用python3 有两种方式
    1、软连接
    2、把安装目录添加到环境变量
    
    添加软连接：
    ln -s /usr/local/python3/bin/python3 /usr/bin/python3
    ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
    
    添加环境变量：
    cat >> /etc/profile  << "EOF"

    # pyrhon3
    export PYTHON_HOME=/usr/local/python3
    export PATH=$PYTHON_HOME/bin:$PATH
    
    EOF
    
    加载文件：
    source /etc/profile
    

#### 验证

以上步骤都操作完成后

使用  python3 命令查看是否能够进入安装的版本环境，如下：

    Python 3.9.6 (default, Jun 29 2021, 09:57:53) 
    [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> 

如果正确进入，说明已经正常安装

后面需要运行python3的脚本就可以使用python3进行运行

eg: python3 test.py

### 后记

本文主要是记录编译安装python3的过程

如果有任何疑问或建议欢迎讨论

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
