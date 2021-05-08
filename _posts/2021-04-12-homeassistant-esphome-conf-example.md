---
layout: post
title: "homeassistant系列之esphome常用配置"
subtitle: '通过esphome配置esp模块-实用配置举例'
author: "chaoxiaodi"
header-style: text
tags:
  - homeassistant
  - esphome
  - esp8266
  - esp32
  - 智能家居
  - 传感器
---

### 前言

莫名其妙的就开始了homeassistant的文章

上一篇文章主要介绍了esphome的基础配置

并没有具体的实例应用

连接在这：[esphome基础配置](https://blog.chaoxiaodi.tech/2021/04/09/homeassistant-esphome-conf/)

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
 
#### 温湿度传感器

首先从最简单的开始

esphome的官网也有温湿度传感器的配置

我测试的时候还是用的DHT11模块

使用温湿度传感器主要有三个引脚

正极/接地/信号

正极接esp8266的3.3v输出引脚

接地接esp8266的GND引脚

信号接esp8266的io口

注意：esp8266不是所有引脚都可以使用，要参考具体引脚说明

配置如下：

    # Example configuration entry
    sensor:             # 表明是传感器模块
      - platform: dht   # dht平台
        pin: D2         # esp8266的引脚编号
        temperature:    # 温度
          name: "Living Room Temperature"   # 命名
        humidity:       # 湿度
          name: "Living Room Humidity"      # 命名
        update_interval: 60s       # 更新频率
        
    提示： name 不要使用中文会有问题；哪怕用容易识别的拼音也不要用中文
    
#### 二进制传感器

大多数传感器都可以归类为二进制传感器

比如：红外人体感应、触摸、刷卡等

只要信号值除了1就是0的都可以用这个模块

配置如下：
    
    # Example configuration entry
    binary_sensor:
      - platform: gpio
        pin: D2
        name: "Living Room Light"

    只要有一个名字和输入引脚的配置就可以简单的完成


#### 输出配置

上面的几个配置基本都是关于输入的

下面介绍一个输出的实例

通过继电器控制灯

    # Example configuration entry
    output:
      - platform: gpio
        pin: D1
        id: relay_light
        
    用D1 引脚接继电器的信号控制线就可以实现通过 继电器控制灯
    

#### 实际应用

在实际测试过程中

发现如果想更好的利用esp8266 和ha

下面可能是一种比较好的方法

这个方法可以最小的改动电路去进行改造

通过把esp8266+继电器做成小的模块放入86的暗盒里

然后原有的灯线路接到继电器

灯的开关接到esp8266的输入引脚

这样不管是通过开关，还是通过ha操作，ha的状态都能正常的显示；

这个思路来自论坛里的一个大佬的想法

我本来的思路是把继电器当作双控开关与物理的双控开关去进行接入

但是那样的话就没法判断双控开关的结果

具体配置如下：

esp8266的引脚13接原开关的负极

esp8266的引脚4是焊接到到继电器的信号控制

这样配置就可以实现对原有开关的改造

    esphome:
      name: livingroom_light
      platform: ESP8266
      board: esp01_1m
    
    wifi:
      ssid: "wifissid"
      password: "wifipwd"
    
    
    web_server:
      port: 80
    captive_portal:
    
    # Enable logging
    logger:
    
    # Enable Home Assistant API
    api:
      password: "12345679"
    
    ota:
      password: "12345679"
    binary_sensor:
      - platform: gpio
        pin:
          number: 13
          mode: INPUT_PULLUP
          inverted: True
        name: "livingroom_light_button_0"
        on_state:
          then:
          - light.toggle: livingroom_light 
    output:
      - platform: esp8266_pwm
        pin: 4
        frequency: 1000 Hz
        id: livingroom_light_p
        inverted: true
    light:
      - platform: binary
        name: "livingroom light"
        output: livingroom_light_p   
        id: livingroom_light  



### 后记

esphome是一个相当强大的开源工具

还有更多更有趣更强大的玩法

如果有感兴趣或好的想法欢迎一起讨论分享

这里放一个上面提到的论坛

里面有不少接入各种设备的案例和教程

[瀚思彼岸](https://bbs.hassbian.com/)

Q：594934249



---我是超小弟·一名不误专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
