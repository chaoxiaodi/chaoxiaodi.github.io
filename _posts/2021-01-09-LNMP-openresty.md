---
layout: post
title: "LNMP系列 - openresty 安装"
subtitle: 'openresty 编译安装'
author: "chaoxiaodi"
header-style: text
tags:
  - lnmp
  - nginx
  - openresty
  - debian
---

### 前言

作为一名运维人员在刚开始的时候永远逃不出 `LNMP` OR `LNMT` 的折磨

更早一点的可能还有 `LAMP`

LNMP: 即 `linux` `nginx` `mysql` `php`

LNMT: 即 `linux` `nginx` `mysql` `tomcat`

A: 即 `Apache httpd`

这篇文章只记录下工作过程中通过编译安装 `openresty` 的一些知识点

### 关于 `nginx` 与 `openresty`

nginx: [官网介绍](http://nginx.org/en/)

    nginx [engine x] is an HTTP and reverse proxy server,
    a mail proxy server, and a generic TCP/UDP proxy server, 
    originally written by Igor Sysoev.
    ---
    nginx [engine x] 是一个 HTTP 和反向代理服务器,
    一个 邮件服务器 和一个通用的 TCP/UDP 代理服务器，
    最初由俄罗斯的 Igor Sysoev 编写而成.
    
openresty: [官网介绍](http://openresty.org/cn/)

    OpenResty® 是一个基于 Nginx 与 Lua 的高性能 Web 平台，
    其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。
    用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。
    
    OpenResty® 通过汇聚各种设计精良的 Nginx 模块（主要由 OpenResty 团队自主开发），
    从而将 Nginx 有效地变成一个强大的通用 Web 应用平台。
    这样，Web 开发人员和系统工程师可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块，
    快速构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。
    
    OpenResty® 的目标是让你的Web服务直接跑在 Nginx 服务内部，
    充分利用 Nginx 的非阻塞 I/O 模型，不仅仅对 HTTP 客户端请求,
    至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。

### 一些常见习惯

系统盘容量: 40-60 GB

数据盘容量: 500G\1T\2T 

个人习惯把单独添加的数据盘挂在到 `/data` 下,并使用以下文件夹命名；

    apps: 放各个应用
    backup: 放备份
    src: 放安装包
    scripts: 放一些脚本

后面一些文章也会使用此目录结构

### 编译安装 `openresty`

openresy 官网的对于安装也提供了对应的文档，对一些常见的linux发行版也提供的相对应的安装包。

本文采用源码编译的方式进行安装,把 `openresty` 安装到 `/data/apps/openresty/` 下

#### 下载 `openresty`

[download openresty](http://openresty.org/cn/download.html)

    # 进入放安装包的目录,下载最新的安装包
    cd /data/src/
    wget https://openresty.org/download/openresty-1.19.3.1.tar.gz
    
#### 创建用户与用户组

    groupadd www
    useradd www -g www -s /bin/false -M

#### 解压

    tar -zxvf openresty-1.19.3.1.tar.gz

#### 配置安装

    cd openresty-1.19.3.1
    
    ./configure \
    --prefix=/data/apps/openresty \ # 指定目录
    --user=www \                    # 指定用户
    --group=www \                   # 指定用户组
    --with-threads \                # 启用线程支持
    --with-file-aio \               # 启动 aio 文件支持（难道是异步？没有深入研究）
    --with-http_v2_module \         # http2 
    --with-http_ssl_module \        # ssl
    --with-http_realip_module \     # 获取客户端真实 IP    
    --with-http_addition_module \   # 响应之前或者之后追加文本内容
    --with-http_gzip_static_module \ 
    --with-stream_realip_module \
    --with-ipv6
    
    
#### make && make install

    make -j2 # 如果是2核机器可以使用 2 加快编译速度


#### 依赖报错
    apt install make cmake openssl libssl-dev libpcre3 libpcre3-dev gcc zlib1g-dev
    
    yum install make cmake openssl openssl-devel pcre-devel zlib-devel gcc 
    
    
#### 添加软连接到环境变量目录
    ln -s /data/apps/openresty/nginx/sbin/nginx /usr/bin/nginx
    
#### 查看通过上述安装后结果

除了指定的模块外,还会安装一些默认的模块

本次安装后执行 nginx -V 后的结果

    nginx -V
    
    nginx version: openresty/1.19.3.1
    built by gcc 8.3.0 (Debian 8.3.0-6) 
    built with OpenSSL 1.1.1d  10 Sep 2019
    TLS SNI support enabled
    configure arguments: --prefix=/data/apps/openresty/nginx --with-cc-opt=-O2
    --add-module=../ngx_devel_kit-0.3.1 --add-module=../echo-nginx-module-0.62 
    --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2 --add-module=../set-misc-nginx-module-0.32 
    --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.08 
    --add-module=../srcache-nginx-module-0.32 --add-module=../ngx_lua-0.10.19 --add-module=../ngx_lua_upstream-0.07 
    --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 
    --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 
    --add-module=../redis-nginx-module-0.3.7 --add-module=../rds-json-nginx-module-0.15 
    --add-module=../rds-csv-nginx-module-0.09 --add-module=../ngx_stream_lua-0.0.9 
    --with-ld-opt=-Wl,-rpath,/data/apps/openresty/luajit/lib --user=www --group=www --with-threads 
    --with-file-aio --with-http_v2_module --with-http_ssl_module --with-http_realip_module 
    --with-http_addition_module --with-http_gzip_static_module --with-stream_realip_module --with-ipv6 
    --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module

### 启动配置

在没有 `systemctl` 之前,一般都是写一个脚本放到 `/etc/init.d/` 下 

现在基本都是去配置一个 `systemctl` 的配置文件

    vi /usr/lib/systemd/system/nginx.service
    
    [Unit]                                        
    Description=nginx - high performance web server            
    After=network.target remote-fs.target nss-lookup.target   
    
    [Service]                                                                       
    Type=forking                                                                        
    PIDFile=/data/apps/openresty/nginx/logs/nginx.pid                             
    ExecStartPre=/data/apps/openresty/nginx/sbin/nginx -t -c /data/apps/openresty/nginx/conf/nginx.conf  
    ExecStart=/data/apps/openresty/nginx/sbin/nginx -c /data/apps/openresty/nginx/conf/nginx.conf     
    ExecReload=/data/apps/openresty/nginx/sbin/nginx -s reload                                             
    ExecReload=/bin/kill -s HUP $MAINPID
    KillSignal=SIGQUIT
    TimeoutStopSec=5
    KillMode=process
    PrivateTmp=true
    
    [Install]
    WantedBy=multi-user.target

### 启动\停止\重载

    systemctl start nginx
    systemctl stop nginx
    systemctl restart nginx
    systemctl reload nginx
    systemctl status nginx
    systemctl enable nginx
