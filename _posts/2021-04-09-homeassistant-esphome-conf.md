---
layout: post
title: "homeassistant系列之esphome基础配置"
subtitle: '通过esphome配置esp模块-基础配置'
author: "chaoxiaodi"
header-style: text
tags:
  - homeassistant
  - esphome
  - esp8266
  - esp32
  - 智能家居
---

### 前言

莫名其妙的就开始了homeassistant的文章

本文主要是介绍esphome的一些常见配置

[ESPHome官网](https://www.esphome.io/)

首先贴一段`esphome`官网的介绍
    
    ESPHome is a system to control your ESP8266/ESP32 by simple yet powerful configuration files and control them remotely through Home Automation systems.
    
    ESPHome 是一个系统，可以通过简单但功能强大的配置文件来控制你的 ESP8266/ESP32，并通过家庭自动化系统对其进行远程控制
    
 关于ESPHome的安装可以自行查阅相关资料
 
 下面就开始进行一些配置的介绍
 
 ESPHome的配置是使用YAML格式
 
 YAML 是要求使用严格的缩进与键值对格式进行配置
 
 独立模块的首行必须顶格
 
 下面依次缩进
 
 配置项必须为 key: value
 
 冒号后面1个空格
 
 #### 核心配置
 
    说是核心配置也不太准确
    应该算是不可缺少的配置，或者每次配置都必须要正确配置的配置
    
    如果你安装了web页面，通常这部分配置不需要配置
    当然可以自己修改
    
    esphome: # 首行配置
      name: livingroom_light # 节点的名称，在整个网络环境里应该唯一 只能包含小写字符、数字、下划线
      platform: ESP8266 # 平台名称 可以是ESP8266/ESP32
      board: esp01_1m # 板子的具体型号，尽量与真实型号保持一致，可能会影响引脚别名或内部设置
 
#### 网络配置

    一般开始配置的第一项工作就是进行联网设置
    ESP芯片的网络还是比较稳定的
    
    配置格式如下：
    
    # Example configuration entry
    wifi:
      ssid: wifissid
      password: wifipassword

    # 如果想手工设置IP可以添加下面的配置
    # 建议使用静态IP，这样既方便进行管理又能缩短wifi的连接时间

      # Optional manual IP
      manual_ip:
        static_ip: 10.0.0.42
        gateway: 10.0.0.1
        subnet: 255.255.255.0
        dns1: 114.114.114.114
        
    如果想连接多个网络可以使用如下配置
    
    wifi:
      networks:
        - ssid: wifi1
          password: wifi2password
        - ssid: wifi2
          password: wifi2password


#### OTA下载

第一次配置完成后需要通过物理连接

把固件刷到ESP的板子上

可以通过设置ota功能来开启空中下载

即通过wifi直接在线升级固件

这样大大方便了固件更新配置

    ota:
      password: "12345678"
      
#### Home Assistant API

    通过使用此配置可以方便的添加板子到Home Assistant 里
    
    api:
      password: "12345678"     

#### 一些其他可选配置

    web_server:
      port: 80
    captive_portal:

    # Enable logging
    logger:


### 后记

本篇文章只记录一些使用ESPHome的基础配置

没有关于接入其他具体模块或功能的配置

下篇文章会写一些常见的接入功能



---我是超小弟·一名不误专业的秃头运维---

博客：blog.chaoxiaodi.tech

github：https://github.com/chaoxiaodi

微信公众号：老骥不伏枥只是近黄昏






