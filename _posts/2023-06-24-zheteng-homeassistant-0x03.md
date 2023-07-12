---
layout: post
title: "homeassistant系列之另类天气玩法"
subtitle: '折腾系列-home-assistant智能家居中枢-0x03'
author: "chaoxiaodi"
header-style: text
tags:
  - home-assistant
  - homeassistant
  - rest
  - 智能家居
  - 智障家居
  - 智能开关
  - weather
  - 折腾记录
---

## 前言

想要更好的玩转智能家居

天气信息可以说是必不可少的一项集成

不管是根据天气信息发送预警信息

还是根据天气调整灯光、窗帘、空调等

论坛里也已经有了不少非常厉害的天气集成插件

和风 彩云 不需要key的nmc等

这篇文章通过一种另辟蹊径的玩法接入下天气信息

## 实现

通过刷论坛找到了一个除nmc之外的另一个不需要key就能获取到天气数据的接口

    # citykey 这个是城市代码 通过百度就能搜到一大堆
    https://zhwnlapi.etouch.cn/Ecalender/api/v2/weather?citykey=101091001

### 收集天气信息

冒起这个念头是想到ha是支持rest接入的

本着学习下用法实现简单的接入

接口反馈的信息很多可以根据需要自己修改配置

主要的配置就三项

`json_attributes_path` `value_template` `json_attributes`

`json_attributes_path` 用来定位数据在返回信息中的位置

`value_template` 可以通过模版来获取/处理信息 

`json_attributes_path` 这个配置下的数据 会作为输入 传递给 `value_template`

`value_template` 可以通过 `value` `value_json` 来接收

`value_template` 最后输出的值为sensor的state 不能超过255个字符

如果返回结果是 json 可以进一步使用 模版语法进行处理 参考 提醒项

唯一不足的地方是数组处理起来不太友好

    # configuration.yaml
    rest: !include rest.yaml

    # rest.yaml
    - scan_interval: 600
      resource: https://zhwnlapi.etouch.cn/Ecalender/api/v2/weather?citykey=101091001
      headers: 
        User-Agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"
      sensor:
        - name: zhwnlapi.提醒
          unique_id: zhwnlapi.future_remind
          json_attributes_path: '$'
          value_template: "{{ value_json.future_remind }}"
        # anomaly
        - name: zhwnlapi.异常
          unique_id: zhwnlapi.anomaly
          json_attributes_path: '$.anomaly'
          value_template: "OK"
          json_attributes:
            - desc
            - detail
            - short_desc
        # meta
        - name: zhwnlapi.元数据
          unique_id: zhwnlapi.meta
          json_attributes_path: '$.meta'
          value_template: "OK"
          json_attributes:
            - up_date
            - up_time
            - city
            - citykey
            - upper
        # alarm
        - name: zhwnlapi.预警
          unique_id: zhwnlapi.alarm
          json_attributes_path: '$.alarm'
          value_template: "OK"
          json_attributes:
            - degree
            - city_range
            - type
            - short_title
            - details
            - location
            - short_desc
            - desc
        # forecast15
        - name: zhwnlapi.forecast15.0
          unique_id: zhwnlapi.forecast15.0
          json_attributes_path: '$.forecast15[0]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.1
          unique_id: zhwnlapi.forecast15.1
          json_attributes_path: '$.forecast15[1]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.2
          unique_id: zhwnlapi.forecast15.2
          json_attributes_path: '$.forecast15[2]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.3
          unique_id: zhwnlapi.forecast15.3
          json_attributes_path: '$.forecast15[3]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.4
          unique_id: zhwnlapi.forecast15.4
          json_attributes_path: '$.forecast15[4]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.5
          unique_id: zhwnlapi.forecast15.5
          json_attributes_path: '$.forecast15[5]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.6
          unique_id: zhwnlapi.forecast15.6
          json_attributes_path: '$.forecast15[6]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.7
          unique_id: zhwnlapi.forecast15.7
          json_attributes_path: '$.forecast15[7]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.8
          unique_id: zhwnlapi.forecast15.8
          json_attributes_path: '$.forecast15[8]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.9
          unique_id: zhwnlapi.forecast15.9
          json_attributes_path: '$.forecast15[9]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.10
          unique_id: zhwnlapi.forecast15.10
          json_attributes_path: '$.forecast15[10]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.11
          unique_id: zhwnlapi.forecast15.11
          json_attributes_path: '$.forecast15[11]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.12
          unique_id: zhwnlapi.forecast15.12
          json_attributes_path: '$.forecast15[12]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.13
          unique_id: zhwnlapi.forecast15.13
          json_attributes_path: '$.forecast15[13]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        - name: zhwnlapi.forecast15.14
          unique_id: zhwnlapi.forecast15.14
          json_attributes_path: '$.forecast15[14]'
          value_template: "OK"
          json_attributes:
            - date
            - uv_level
            - aqi
            - aqi_level
            - aqi_level_name
            - sunrise
            - sunset
            - rainIntensity
            - high
            - low
            - humidity
        # hourforecast
        - name: zhwnlapi.hourfc.0
          unique_id: zhwnlapi.hourfc.0
          json_attributes_path: '$.hourfc[0]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.1
          unique_id: zhwnlapi.hourfc.1
          json_attributes_path: '$.hourfc[1]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.2
          unique_id: zhwnlapi.hourfc.2
          json_attributes_path: '$.hourfc[2]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.3
          unique_id: zhwnlapi.hourfc.3
          json_attributes_path: '$.hourfc[3]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.4
          unique_id: zhwnlapi.hourfc.4
          json_attributes_path: '$.hourfc[4]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.5
          unique_id: zhwnlapi.hourfc.5
          json_attributes_path: '$.hourfc[5]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.6
          unique_id: zhwnlapi.hourfc.6
          json_attributes_path: '$.hourfc[6]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.7
          unique_id: zhwnlapi.hourfc.7
          json_attributes_path: '$.hourfc[7]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.8
          unique_id: zhwnlapi.hourfc.8
          json_attributes_path: '$.hourfc[8]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.9
          unique_id: zhwnlapi.hourfc.9
          json_attributes_path: '$.hourfc[9]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.10
          unique_id: zhwnlapi.hourfc.10
          json_attributes_path: '$.hourfc[10]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.11
          unique_id: zhwnlapi.hourfc.11
          json_attributes_path: '$.hourfc[11]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.12
          unique_id: zhwnlapi.hourfc.12
          json_attributes_path: '$.hourfc[12]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.13
          unique_id: zhwnlapi.hourfc.13
          json_attributes_path: '$.hourfc[13]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.14
          unique_id: zhwnlapi.hourfc.14
          json_attributes_path: '$.hourfc[14]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.15
          unique_id: zhwnlapi.hourfc.15
          json_attributes_path: '$.hourfc[15]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.16
          unique_id: zhwnlapi.hourfc.16
          json_attributes_path: '$.hourfc[16]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.17
          unique_id: zhwnlapi.hourfc.17
          json_attributes_path: '$.hourfc[17]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.18
          unique_id: zhwnlapi.hourfc.18
          json_attributes_path: '$.hourfc[18]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.19
          unique_id: zhwnlapi.hourfc.19
          json_attributes_path: '$.hourfc[19]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.20
          unique_id: zhwnlapi.hourfc.20
          json_attributes_path: '$.hourfc[20]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.21
          unique_id: zhwnlapi.hourfc.21
          json_attributes_path: '$.hourfc[21]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.22
          unique_id: zhwnlapi.hourfc.22
          json_attributes_path: '$.hourfc[22]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu
        - name: zhwnlapi.hourfc.23
          unique_id: zhwnlapi.hourfc.23
          json_attributes_path: '$.hourfc[23]'
          value_template: "OK"
          json_attributes:
            - time
            - wthr
            - aqi
            - aqi_level
            - aqi_level_name
            - type_desc
            - wd
            - wp
            - shidu

使用了 `json_attributes` 配置的最后会显示为下图效果

![](/img/haweatherattr.png)


### 展示天气信息

像这样收集起来的信息

展示起来是不能通过那个dashboard的 `weather-forecast` 去添加的

当然了就做不到像和风、彩云看起那么高大上了

除了收集另辟蹊径 展示也另辟蹊径一下

直接在ha中集成一个其他的天气页面

这个在这个接口中也有一个返回

source.link 这个返回里的链接可以直接用到ha里

添加卡片-选择网页-填入 url就成了

那么怎么使用收集起来的数据呢

ha支持的模版语法就够用了

state_attr(实体id, 属性)

如： state_attr('sensor.zhwnlapi_forecast15_0', 'humidity')

表示获取今天的湿度数据

下面提供一份通过模版语法使用收集起来数据的作业

    # 添加卡片选择实体
    type: entities
    entities:
      - entity: sensor.zhwnlapi_yuan_shu_ju
        type: attribute
        name: 省份
        attribute: upper
      - entity: sensor.zhwnlapi_yuan_shu_ju
        type: attribute
        name: 城市
        attribute: city
      - entity: sensor.zhwnlapi_yuan_shu_ju
        type: attribute
        name: 更新时间
        attribute: up_time
      - type: divider
      - entity: sensor.zhwnlapi_ti_xing
        name: 概况
      - entity: sensor.zhwnlapi_yi_chang
        type: attribute
        name: 异常
        attribute: short_desc
      - entity: sensor.zhwnlapi_yi_chang
        type: attribute
        name: 异常
        attribute: desc
      - entity: sensor.zhwnlapi_yi_chang
        type: attribute
        name: 异常
        attribute: detail
      - type: divider
      - entity: sensor.zhwnlapi_yu_jing_2
        type: attribute
        name: 预警
        attribute: short_title
      - entity: sensor.zhwnlapi_yu_jing_2
        type: attribute
        name: 预警
        attribute: short_desc
      - entity: sensor.zhwnlapi_yu_jing_2
        type: attribute
        name: 预警
        attribute: details
      - entity: sensor.zhwnlapi_yu_jing_2
        type: attribute
        name: 预警
        attribute: desc

    # 添加卡片选择markdown
    type: markdown
    content: |-
      {\\% set ids = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14] %}
      {\\% set forecase15 = [
      'date','uv_level','aqi','aqi_level','aqi_level_name','sunrise','sunset',
      'rainIntensity','high','low','humidity'
      ] %}
      |日期|紫外线强度|空气质量指数|空气质量级别|空气质量描述|日出|日落|降雨强度|最高温度|最低温度|湿度|
      |:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|:----|
      {\\% for id in ids %}
        {\\% set sid = 'sensor.zhwnlapi_forecast15_' ~ id %}
        {\\%- for attr in forecase15 -%}
      |{{ state_attr(sid, attr) }}
        {\\%- endfor -%}
      |
      {\\% endfor %}
    title: 15日天气预报


    type: markdown
    content: >-
      {\\% set ids = [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23]
      %}

      {\\% set hourfc = [

      'time','wthr','aqi','aqi_level','aqi_level_name','type_desc','wd', 'wp',
      'shidu'

      ] %}

      |时间|温度|空气质量指数|空气质量级别|空气质量描述|天气|风向|风力|湿度|

      |---|---|---|---|---|---|---|---|---|

      {\\%- for id in ids %}
        {\\% set sid = 'sensor.zhwnlapi_hourfc_' ~ id %}
      |
        {\\%- for attr in hourfc -%}
      {{ state_attr(sid, attr) }}|
        {\\%- endfor -%}
      {\\% endfor%}
    title: 24小时天气预报

通过实体的card推荐接入一些独立的信息

一些可以循环的信息使用markdown会比较好

同时这里也希望有大佬能指点下 为啥想用个table格式显示 没有成功

![另辟蹊径的天气接入](/img/haweatherrestapi.png)

## 参考

[home-assistant 官网](https://www.home-assistant.io/)

[论坛 官网](https://bbs.hassbian.com/)

[nmcweather集成](https://github.com/chaoxiaodi/homeassistant.components.weather.Weathernmc)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)

