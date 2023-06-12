---
layout: post
title: "homeassistant系列之再探esphome"
subtitle: '折腾系列-home-assistant智能家居中枢-0x02'
author: "chaoxiaodi"
header-style: text
tags:
  - home-assistant
  - homeassistant
  - esphome
  - 智能家居
  - 智障家居
  - 智能开关
  - espdiy
  - 折腾记录
---

## 前言

距离上次探索esphome已经有很长一段时间了

最近为了使用群友diy的基于`esp32-c3`做的智能开关

又进一步学习了下esphome

这片文章不谈那些基础的配置 比如wifi配置、gpio输入输出配置 api ota等基础配置

聊一下关于`esphome`更优雅的使用方式

以及更好的体现`esphome`的强大

再次附上官网对于`esphome`的介绍

    ESPHome is a system to control your ESP8266/ESP32 by simple yet powerful configuration files and control them remotely through Home Automation systems.

## 更好的使用esphome

### 官网文档推荐

在官网的左侧主要列出了 esphome 支持的各种组件及各种配置方式

在首页的 Next steps 里的 FAQ and Tips、Automations、Configuration types、DIY Examples 中有着详细的esphome的高级用法

Automations and Templates 尤其是这个主题下的内容 是进阶esphome的必读内容

下面几个小节 大部分都是 基于这个主题下的内容完成的

### 结构化代码

这一部分主要是 利用 `!include` 官网文档中 给出了一些关于 include的示例

关于配置可以看下 官网链接 [yaml-insertion-operator](https://esphome.io/guides/configuration-types.html#yaml-insertion-operator)

相信如果有接触过esphome的朋友

就会发现 当有多种esphome硬件的时候 

或者当一类硬件有多个配置的时候

比如 1键开关/2键开关/3键开关/4键开关 甚至变态的9键开关(一个用完esp32-c3所有io口的开关)

当出现这种情况的时候就会代码就会出现相当多的重复

包括基础配置 比如上面提到的 wifi api ota等

esphome在很久之前的版本就已经官方默认支持 `secrets.yaml` 了

这个配置其实已经可以简化不少配置动作了

其实关于 `!include` 主要就是两类

    # 被引用的是多项不同内容
    当需要把文件内容当作key/value引用的时候 在对应缩进 使用 <<: !include filename

    # 被引用的是一项具体内容
    当需要把文件内容当作列表中的一项的时候 在对应缩进 使用 - !include filename

更好的结构化能够更好的更方便的进行维护

比如 我有一个变态的9键开关 我想实现 按对应按键对应的led灯闪烁一次作为反馈

那么按照正常写法

    ### --- 当按键的时候点亮RGB灯 --- ###
    # 每个按键下都要写这么一段重复代码
    # 而且如果想要修改里面的配置并同步的话就需要修改9次
    on_press:
      then:
        - light.addressable_set:
            id: gpio_10
            range_from: 2
            range_to: 2
            color_brightness: 100%
            # 下面是三行随机颜色
            red: !lambda |-
              return float(random(100) / 100.0);
            green: !lambda |-
              return float(random(100) / 100.0);
            blue: !lambda |-
              return float(random(100) / 100.0);
            # led灯的效果 要对应灯效果里配置的
        # 让灯熄灭
        - light.turn_off: gpio_10
        - switch.toggle: ${device_name}_relay_3

按照coding的习惯 这段亮灯的代码完全可以抽象成一个脚本，然后传递对应参数就可以了

这个在下面进行介绍

### 变量替换

变量替换可以更加清晰的维护配置

比如 把io针脚与对应功能进行对应 `key_1: GPIO9` `relay_pin_1: GPIO0`

这样在对应的配置位置就可以直接使用 `key_1` 代表 `GPIO9` 了 

可以类比为 域名与ip地址的对应关系 用更方便记忆的名字去对应一个配置

变量替换举例 foo: bar

name: $foo 这样配置说明 name 最后的值为 bar

name: ${foo}_1 这样配置说明 name 最后值为 bar_1 当需要把变量替换和其他字符连接起来时需要用{}把变量包裹起来

tips: 变量可以在多层次引用文件中使用

### 条件判断

作为编程三大结构中使用率最高的条件判断

在esphome当然也是必不可少的

当然了 循环在esphome也是支持

在大部分需要执行动作的时候都可以进行条件判断

    on_...:
      then:
        - if:
            condition:
              # 单条件为真
              binary_sensor.is_on: binary_sensor1
            then:
              # xxxx
            else:
              # xxxx

        - if:
            condition:
              and:
                # 两个条件同时为真才执行then下动作
                - binary_sensor.is_on: binary_sensor1
                - binary_sensor.is_on: binary_sensor2

        - if:
            condition:
              
              or:
                # 两个条件只要有一个为真就执行then下动作
                - binary_sensor.is_on: some_binary_sensor
                - binary_sensor.is_on: other_binary_sensor
            # ...

        - if:
            condition:
              not:
                # 条件为假的时候执行 then 下动作
                binary_sensor.is_off: some_binary_sensor

论坛里有一个diy开关的 固件里配置里使用了大量的条件判断

### 脚本

使用脚本功能可以简化代码，也可以让esphome实现更多自定义功能

比如上面我提到的，把按键亮灯的代码使用了脚本实现，每个按键直接调用就可以了

如果只是调用esphome中组件的支持的动作其实跟配置到每个按键下几乎是一样的

可以对比下跟刚才按键下的代码 就是把要亮哪个灯珠作为参数提取出来然后再塞进去

    # 根据按键 点亮对应的led灯珠
    # 每次单击按下按键 对应位置led闪烁一次作为状态反馈
    # 接受两个参数
    # 从哪个灯珠到那个灯珠亮 from/to
    id: turn_on_led_by_key
    parameters:
      led_num_from: int
      led_num_to: int
    then:
      light.addressable_set:
        id: led_light
        color_brightness: 100%
        range_from: !lambda return led_num_from;
        range_to: !lambda return led_num_to;
        red: !lambda |-
          return float(random(100) / 100.0);
        green: !lambda |-
          return float(random(100) / 100.0);
        blue: !lambda |-
          return float(random(100) / 100.0);

代码执行的话也是非常简单的配置

id 对应上 脚本定义的id 参数对应 需要的参数就可以了

    - script.execute: 
        id: turn_on_led_by_key
        led_num_from: 3
        led_num_to: 3

有了脚本功能 搭配结构化代码 可以使`esphome`更加实用

## 后记

附一份我的esphome配置结构参考

[esphome-config](https://github.com/chaoxiaodi/esphome-config)

推荐几个espdiy的资源

[论坛里大佬开源的人体存在](https://github.com/liwei19920307/ESPMMW)

群里也有好多diy智能开关的

我的智能开关也是跟群友交易的

[86开关模块V2.0,开启彩灯模式](https://bbs.hassbian.com/forum.php?mod=viewthread&tid=21097&extra=page%3D1%26filter%3Dtypeid%26typeid%3D61)

[一个我个人认为颜值天花板的智能开关 肤感钢化玻璃面板](https://oshwhub.com/xtjunv/m10-zhi-neng-kai-guan)

如果有一定的动手能力 那么diy将会非常有趣


## 参考

[home-assistant 官网](https://www.home-assistant.io/)

[esphome 官网](https://esphome.io/)

[论坛 官网](https://bbs.hassbian.com/)

[esphome-config](https://github.com/AlexMekkering/esphome-config)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
