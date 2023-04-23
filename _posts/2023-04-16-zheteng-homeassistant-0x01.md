---
layout: post
title: "homeassistant系列之RS485与modbus"
subtitle: '折腾系列-home-assistant智能家居中枢-0x01'
author: "chaoxiaodi"
header-style: text
tags:
  - home-assistant
  - homeassistant
  - rs485
  - modbus
  - 智能家居
  - 智障家居
  - 智能电表
  - 继电器
  - 折腾记录
---

### 前言

本篇文章记录下home-assistant 通过 modbus集成 与 rs485硬件 通信的测试配置过程

相关硬件

rs485继电器模组

正泰DDSU666智能电表

tcp转rs485模块(汉枫)

#### 关于modbus

    Modbus is a data communications protocol originally published by Modicon (now Schneider Electric) in 1979 for use with its programmable logic controllers (PLCs). Modbus has become a de facto standard communication protocol and is now a commonly available means of connecting industrial electronic devices.

    Modbus是一种串行通信协议，是Modicon公司（现在的施耐德电气 Schneider Electric）于1979年为使用可编程逻辑控制器（PLC）通信而发表。Modbus已经成为工业领域通信协议事实上的业界标准，并且现在是工业电子设备之间常用的连接方式。
    
    -- 来自 wikipedia

#### 关于rs485

    RS-485, also known as TIA-485(-A) or EIA-485, is a standard defining the electrical characteristics of drivers and receivers for use in serial communications systems. Electrical signaling is balanced, and multipoint systems are supported.

    EIA-485（过去叫做RS-485或者RS485[1]）是隶属于OSI模型物理层的电气特性规定为2线、半双工、平衡传输线多点通信的标准。

    -- 来自 wikipedia

#### 个人理解

modbus 是一种通信协议，类似于网络中的tcp/ip，而rs485是一用硬件接口标准。类似于双绞线rj45接口、同轴电缆接口等

同样的之前使用过的还有rs23，这个串口之前用的最多的就是调试网络设备的时候。

通过usb转rs232的设备插上网络设备的口就可以直接进行调试了

modbus 可以在同一条线路上允许有多个硬件

那么必然要对硬件有一个编号，也就是同一串行线路上的硬件要有唯一的一个地址

modbus 提供了一些常见的读写操作功能码

modbus常用16进制表示

modbus 采用CRC校验

一个modbus请求报文结构如下

0x01 0x03 0x00 0x05 0x00 0x02 0x0c 0x14

地址 功能码 寄存器起始高字节 寄存器起始低字节 寄存器数量高字节 寄存器数量低字节 CRC校验低字节 CRC校验高字节

上面报文代表

硬件的地址为1 16进制表示为0x01

0x03 功能码读取保持寄存器的值

0x00 0x05 表示寄存器地址 这个地址一半硬件厂家都会提供

0x00 0x02 表示读取几个寄存器 这个数量硬件厂家也会提供 下面配置的时候会有解释

0x0c 0x14 这个一般调试软件会自动追加校验

详细了解modbus可以查看[modbus协议手册](/attachment/about-modbus-rtu.pdf)

### 测试过程

工欲善其事、必先利其器

首先是硬件

要想调试rs485那么必须得有对应的转接头

rs485一般有AB两根线，可以买上文提到的usb转485接头(大概10元左右)

软件的话可以下载sscom

下载sscom[sscom5.13](http://www.daxia.com/)

还有就是一般生产硬件的厂家会提供可视化的调试软件

可以跟对应的厂家要

#### rs485继电器模组

我买的这个继电器模组

提供了对应的调试软件

同时提供了一份调试说明书

主要是需要说明书中的各个寄存器的地址，以及对应的值代表什么操作

比如我买的这个继电器就是通过 0x00-0x15 对应1-16路继电器

通过向寄存器写入1/0 分别表示开启/关闭

还有就是对模块的一些串口参数进行设置

比如 地址、波特率、数据位、停止位、校验位等

其中地址就是上文提到的 整个线路上唯一的地址 

因为我这是个16路的继电器我选择直接把地址设置为16

波特率这个各个硬件支持的不同，但是要保证同一条线路上的波特率保持一致

下面的智能电表支持有限的几个波特率

为了适配选择了 电表同样支持的波特率 9600

这个设置的参数，后面会在其他的配置文件中用到

#### ddsu666智能电表

这个型号的智能电表所支持的modbus功能码很少

所支持的波特率也很少

所以基本可配置的东西不是特别多

有两点情况需要注意

一个是协议

在homeassistant中配置modbus 可以使用tcp udb rtuovertcp等

上面的继电器模组可以正常使用tcp

但这个电表必须设置为rtuovertcp才能正常读取数据

另一个是返回数据

由于我刚开始测试是从继电器开始的

所以当我使用tcp协议去电表获取数据的时候

一直提示没有正常返回数据

然后就又用串口调试工具去读取对应的寄存器

获取到的是两段16进制的数据

比如我读取的是电压，返回的数据是

4363 B333

4364 xxxx

前面一段基本是稳定的不变的
后面几乎每次都不一样

我猜测应该是取到数据了

但是无论用进制怎么换算都得不到正确的值

经过群里大佬指点

还有一个什么 IEEE 745数据转换

4363 B333 -> 227.6999969482422

[在线进制转换](https://lostphp.com/hexconvert/)

在确认能够正常拿到数据后

再次回到homeassistant去看对应的配置

然后就发现了上面协议支持的问题

[DDSU666 说明书](https://github.com/liwei19920307/ESP485/tree/main/doc/DDSU666.pdf)

#### tcp转rs485模块

现在x宝上搜这种转换模块

能搜到真的不少

有人、塔石、汉枫、亿百特等等

我买了其中的汉枫和亿百特

最后用了汉枫

主要是汉枫直接可以通过网页就能进行配置管理

对懒人比较友好

亿百特还需要下载他们提供的专用软件才能配置

针对模块的配置主要有以下几点

修改管理地址、保证跟家庭局域网(ha)在一个网段

修改串口设置 就上面提到的波特率那些

修改tcp 端口 后面homeassistant 使用

最后提一句

个人感觉这个模块技术含量不是很高

没必要花大钱

#### home-assistant modbus接入

modbus相关的配置可以查看文档
[home-assistant modbus 集成](https://www.home-assistant.io/integrations/modbus/)

配置正确的device-class 可以在页面上得到一个预设的icon
[传感器支持的device-class](https://www.home-assistant.io/integrations/sensor/#device-class)

home-assistant 官网提供了不少的配置案例

主要参考platform sensor的配置使用

对应的可以选择对应的数值属于哪个类别

home-assistant真的强

各种想到想不到的数据类型几乎都包含了

    # 主要的配置文件就这几行
    # host 设置为上面提到的模块的管理地址
    # type 就是上面踩坑的协议 建议设置为这个 兼容性更高
    # port 也是上面配置的模块的端口
    - name: hanfeng-pe11
      type: rtuovertcp
      host: x.x.x.x
      port: 502

      # 关于模块的配置 主要就是配置一个上面设置的地址
      # 以及配置寄存器的地址
      switches:
        - name: Switch01
          slave: 16
          address: 0x00

      # 根据说明书 正确填写寄存器地址
      # 寄存器长度
      # 数据类型
      # 这里要提到的一点是，上面踩的数据转换的坑 不需要单独配置就可以正常显示
      sensors:
        # A 相电压
        - name: dian_ya
          unit_of_measurement: V # 单位
          slave: 1               # modbus硬件地址
          count: 2               # 寄存器长度
          address: 0x2000        # 寄存器地址
          data_type: float32     # 数据类型
          device_class: voltage  # 设备类型，会根据设备类型提供一个预设的icon
          input_type: holding    # 寄存器类型 默认也是这个 具体可以看官网
          precision: 1           # 数据精度 (保留几位小数)

#### 配置备份

下面是可以直接抄作业的配置

留一个备份

    - name: xxx
      type: rtuovertcp
      host: x.x.x.x
      port: 502
      switches:
        - name: Switch01
          slave: 16
          address: 0x00

        - name: Switch02
          slave: 16
          address: 0x01

        - name: Switch03
          slave: 16
          address: 0x02

        - name: Switch04
          slave: 16
          address: 0x03

        - name: Switch05
          slave: 16
          address: 0x04

        - name: Switch06
          slave: 16
          address: 0x05

        - name: Switch07
          slave: 16
          address: 0x06

        - name: Switch08
          slave: 16
          address: 0x07

        - name: Switch09
          slave: 16
          address: 0x08

        - name: Switch10
          slave: 16
          address: 0x09

        - name: Switch11
          slave: 16
          address: 0x10

        - name: Switch12
          slave: 16
          address: 0x11

        - name: Switch13
          slave: 16
          address: 0x12

        - name: Switch14
          slave: 16
          address: 0x13

        - name: Switch15
          slave: 16
          address: 0x14

        - name: Switch16
          slave: 16
          address: 0x15
      sensors:
        # A 相电压
        - name: dian_ya
          unit_of_measurement: V
          slave: 1
          count: 2
          address: 0x2000
          data_type: float32
          device_class: voltage
          input_type: holding
          precision: 1
        # A 相电流
        - name: dian_liu
          unit_of_measurement: A
          slave: 1
          count: 2
          address: 0x2002
          data_type: float32
          device_class: current
          input_type: holding
          precision: 3
        # 瞬时总有功功率
        - name: you_gong_gong_lv
          unit_of_measurement: W
          slave: 1
          count: 2
          address: 0x2004
          data_type: float32
          device_class: power
          input_type: holding
          precision: 1
        # 瞬时总无功功率
        - name: wu_gong_gong_lv
          unit_of_measurement: var
          slave: 1
          count: 2
          address: 0x2006
          data_type: float32
          device_class: power
          input_type: holding
          precision: 1
        # 瞬时总视在功率
        - name: shi_zai_gong_lv
          unit_of_measurement: VA
          slave: 1
          count: 2
          address: 0x2008
          data_type: float32
          device_class: power
          input_type: holding
          precision: 1
        # 总功功率因数
        - name: gong_lv_yin_shu
          # unit_of_measurement: V
          slave: 1
          count: 2
          address: 0x200A
          data_type: float32
          device_class: power_factor
          input_type: holding
          precision: 3
        # 电网频率
        - name: dian_wang_pin_lv
          unit_of_measurement: Hz
          slave: 1
          count: 2
          address: 0x200E
          data_type: float32
          device_class: frequency
          input_type: holding
          precision: 2
        # 有功总电能
        - name: zong_dian_neng
          unit_of_measurement: kWh
          slave: 1
          count: 2
          address: 0x4000
          data_type: float32
          device_class: energy
          input_type: holding
          precision: 2

### 后记

已经初步完成了对485硬件的使用需求

后面针对数据进行展示

针对数据做自动化还需要进一步研究

页面展示效果

![](/img/zheteng-0x01-485relay.png)
![](/img/zheteng-0x01-ddsu666.png)

### 参考

[home-assistant 官网](https://www.home-assistant.io/)

[esphome 官网](https://esphome.io/)

[论坛 官网](https://bbs.hassbian.com/)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
