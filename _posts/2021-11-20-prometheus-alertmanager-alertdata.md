---
layout: post
title: "prometheus alertmanager 告警/恢复 信息体"
subtitle: '记录学习prometheus alertmanager'
author: "chaoxiaodi"
header-style: text
tags:
  - prometheus
  - alertmanager
  - 学习记录
---

### 前言
本文记录下学习使用prometheus alertmanager过程中处理告警信息接收到的数据格式

### 数据格式

#### 告警

    {
      "receiver": "web\\.hook",
      "status": "resolved",
      "alerts": [{
        "status": "resolved",
        "labels": {
          "alertname": "MySQL is down",
          "environment": "xxxxx",
          "instance": "xxxxx",
          "instance_name": "xxxxx",
          "job": "xxxxx",
          "metrics_path": "/scrape",
          "project": "xxxxx",
          "region": "xxxxx",
          "release": "xxxxx",
          "service": "xxxxx",
          "severity": "xxxxx"
        },
        "annotations": {
          "description": "MySQL instance is down on xxxxx\n  VALUE = 0\n",
          "summary": "MySQL down (instance xxxxx)"
        },
        "startsAt": "2021-11-22T05:13:19.764Z",
        "endsAt": "2021-11-22T06:56:19.764Z",
        "generatorURL": "http://xxx:9090/graph?g0.expr=mysql_up+%3D%3D+0&g0.tab=1",
        "fingerprint": "936d920be927dbe6"
      }, {
        "status": "resolved",
        "labels": {
          "alertname": "MySQL is down",
          "environment": "xxxxx",
          "instance": "xxxxx",
          "instance_name": "xxxxx",
          "job": "xxxxx",
          "metrics_path": "/scrape",
          "project": "xxxxx",
          "region": "xxxxx",
          "release": "xxxxx",
          "service": "xxxxx",
          "severity": "xxxxx"
        },
        "annotations": {
          "description": "MySQL instance is down on xxxxx\n  VALUE = 0\n",
          "summary": "MySQL down (instance xxxxx)"
        },
        "startsAt": "2021-11-22T05:13:19.764Z",
        "endsAt": "2021-11-22T06:54:19.764Z",
        "generatorURL": "http://xxxxx:9090/graph?g0.expr=mysql_up+%3D%3D+0&g0.tab=1",
        "fingerprint": "baf9ec548e7b7d8e"
      }],
      "groupLabels": {
        "alertname": "MySQL is down"
      },
      "commonLabels": {
        "alertname": "MySQL is down",
        "environment": "prod",
        "job": "mysql_exporter_targets",
        "metrics_path": "/scrape",
        "project": "xxxxx",
        "region": "xxxxx",
        "release": "xxxxx",
        "service": "xxxxx",
        "severity": "xxxxx"
      },
      "commonAnnotations": {},
      "externalURL": "http://xxxxx:9093",
      "version": "4",
      "groupKey": "{}{alertname='MySQL is down'}",
      "truncatedAlerts": 0
    }


#### 告警恢复
    {
      "receiver": "web\\.hook",
      "status": "resolved",
      "alerts": [{
        "status": "resolved",
        "labels": {
          "alertname": "MySQL is down",
          "environment": "xxxxx",
          "exporter_name": "xxxxx",
          "instance": "xxxxx",
          "instance_name": "xxxxx",
          "job": "xxxxx",
          "metrics_path": "/scrape",
          "project": "xxxxx",
          "release": "xxxxx",
          "service": "xxxxx",
          "severity": "xxxxx"
        },
        "annotations": {
          "description": "MySQL instance is down on xxxxx\n  VALUE = 0\n",
          "summary": "MySQL down (instance xxxxx)"
        },
        "startsAt": "2021-11-22T05:13:19.764Z",
        "endsAt": "2021-11-22T06:45:19.764Z",
        "generatorURL": "http://xxxxx:9090/graph?g0.expr=mysql_up+%3D%3D+0&g0.tab=1",
        "fingerprint": "8fcd4d31dcfbb4eb"
      }, {
        "status": "resolved",
        "labels": {
          "alertname": "MySQL is down",
          "environment": "xxxxx",
          "exporter_name": "mysql-3306",
          "instance": "xxxxx",
          "instance_name": "xxxxx",
          "job": "xxxxx",
          "metrics_path": "/scrape",
          "project": "xxxxx",
          "release": "xxxxx",
          "service": "xxxxx",
          "severity": "xxxxx"
        },
        "annotations": {
          "description": "MySQL instance is down on xxxxx\n  VALUE = 0\n",
          "summary": "MySQL down (instance xxxxx)"
        },
        "startsAt": "2021-11-22T05:13:19.764Z",
        "endsAt": "2021-11-22T06:45:19.764Z",
        "generatorURL": "http://xxxxx:9090/graph?g0.expr=mysql_up+%3D%3D+0&g0.tab=1",
        "fingerprint": "c3bed76621bfd481"
      }, {
        "status": "resolved",
        "labels": {
          "alertname": "MySQL is down",
          "environment": "sandbox",
          "exporter_name": "mysql-3306",
          "instance": "xxxxx",
          "instance_name": "xxxxx",
          "job": "xxxxx",
          "metrics_path": "/scrape",
          "project": "xxxx",
          "release": "xxxx",
          "service": "onebox",
          "severity": "critical"
        },
        "annotations": {
          "description": "MySQL instance is down on xxxxx\n  VALUE = 0\n",
          "summary": "MySQL down (instance xxxxx)"
        },
        "startsAt": "2021-11-22T05:13:19.764Z",
        "endsAt": "2021-11-22T06:45:19.764Z",
        "generatorURL": "xxxxx:9090/graph?g0.expr=mysql_up+%3D%3D+0&g0.tab=1",
        "fingerprint": "a8842c7c8f8354b0"
      }],
      "groupLabels": {
        "alertname": "MySQL is down"
      },
      "commonLabels": {
        "alertname": "MySQL is down",
        "metrics_path": "/scrape",
        "release": "global",
        "severity": "critical"
      },
      "commonAnnotations": {},
      "externalURL": "http://xxxxx:9093",
      "version": "4",
      "groupKey": "{}:{alertname='MySQL is down'}",
      "truncatedAlerts": 0
    }

### 后记


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
