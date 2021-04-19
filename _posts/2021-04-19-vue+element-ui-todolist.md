---
layout: post
title: "基于vue+element-ui实现的纯前端todolist工具"
subtitle: '一个开源的待办任务管理工具网页前端版'
author: "chaoxiaodi"
header-style: text
tags:
  - vue
  - javascript
  - element-ui
  - 前端
  - todolist
---

### 前言
前一段时间使用python实现了一个todolist小工具

文章连接在这[使用python实现一个待办管理小工具](https://blog.chaoxiaodi.tech/2021/03/08/python-todolist/)

因为最近一直在写运维管理平台

也开始写一些前端的东西

想着作为一个学习记录

把用python做好的东西再用前端做一遍

具体的逻辑思路可以参考上篇文章

下面记录下此次实现的过程

#### 思路

由于不想使用数据库

在开始选择怎么保留记录方向上出过错

最开始还是想着用之前的思路

把所有的内容都保存到到文本文件

但是浏览器前端独写文件那是一个不方便

可能是我太菜了

最后还是使用的 localstorage

这个存储好像是最大只能保存5M 

我也没测试过5M能存多少数据

反正应该能存不少

这个只要不清理浏览器数据就会一直存在

你要是问我重装系统还有吗

那你先考虑考虑重装系统浏览器还在不在

#### 整体页面涉及到的element-ui控件

使用vue+element-ui

没有使用vue的脚手架去生成项目

全部引用的对应的js

这样就保证了一个文件能搞定整个功能

不需要去安装一些依赖

但这样也有一个缺陷就是需要联网

需要联网去使用对应的js文件

使用到的控件有：

`el-container` `el-date-picker` `el-input` `el-radio-group`

`el-button`` el-select` `el-table` `el-tree` `el-avatar`` el-form`` el-dialog`

可以说是麻雀虽小、五脏俱全

一般网页常见的控件都有使用到

可以作为小白接触vue+element-ui一个小的学习途径

我也是一个小白通过把这个学习过程记录下来

希望能够帮到有需要的人

#### 体验链接

[vuetodolist](https://blog.chaoxiaodi.tech/todolist.html)

我在我的博客放了一个体验链接

大家可以使用下

### 后记

本篇文章内容不是狠多

因为具体的思路在之前的文章已经提到过

本篇文章只是用另一种语言进行了实现

想了解具体的思路的可以看下那篇文章

欢迎大家交流讨论~

Q：594934249


---我是超小弟·一名不误专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏







