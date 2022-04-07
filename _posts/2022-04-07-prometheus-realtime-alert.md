---
layout: post
title: "prometheus监控系统获取告警信息"
subtitle: '获取prometheus监控中触发的告警信息'
author: "chaoxiaodi"
header-style: text
tags:
  - prometheus
  - alertmanager
  - python
---

### 前言

近一段时间一直在跟prometheus打交道

避免不了的也跟alertmanager打交道

有监控系统必然有告警

这篇文章就是简单的获取触发告警规则的信息


### 思路

#### alertmanager api

    alertmanager api 没有类似其他开源项目的api文档
    官方提供了一个openapi文档
    https://github.com/prometheus/alertmanager/blob/main/api/v2/openapi.yaml

    这个文档刚开始看的时候我是一脸懵逼啊

    后来发现了一个可以解析这个文章的网站
    https://petstore.swagger.io/?url=https://raw.githubusercontent.com/prometheus/alertmanager/main/api/v2/openapi.yaml#/

    但是通过alertmanager api拿到的告警是只有触发了firing之后转发到alertmanger之后的

#### prometheus 特殊 metrics

prometheus 官方文档要求metric命名使用小写字母加下划线命名

但是它自己却暴露了两个全大写字母的metrics

如果需要获取更及时的报警信息

可以使用这两个metrics

ALERTS

ALERTS_FOR_STATE

在官网文档暂时没有找到关于这两个指标的介绍

也是在巧合之下不经意发现的

利用这两个指标对于分析告警有很大帮助

### 后记


如果有获取实时告警信息的需求

可以使用这篇文章中的两个指标

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
