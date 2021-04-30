---
layout: post
title: "shell python javascript分支判断语句汇总"
subtitle: '整理记录常用语言的常用判断语句以便日后查询使用'
author: "chaoxiaodi"
header-style: text
tags:
  - shell
  - python
  - javascript
---

### 前言

在当前的工作中，常用到的语言涉及到 `shell` `python` `javascript`

这三个语言的判断语句写的格式还有些稍微的不同

本文记录下三个语言中常用的分支语句格式

以便学习和使用时查询

#### shell 常用分支格式

shell 中支持 if 与 case  两种分支用法

常用的 if 语句格式如下
    
    # 一层判断
    if 条件 ; then
      代码
      代码
    if
    
    # 两层判断
    if 条件 ; then
      代码
      代码
    else
      代码
      代码
    if
    
    # 多层判断；可以只用多个elif
    if 条件 ; then
      代码
      代码
    elif
      代码
      代码
    else
      代码
      代码
    if

shell 中 case 常用的格式如下

    # case 与 esac 配对使用
    # 判断第一个参数，并根据参数执行对应操作，操作可以有多个
    
    case "$1" in
      "start")
        echo "start"
        ;;
      "stop")
        echo "stop"
        ;;
      "restart")
        echo "restart"
        ;;
      *)
        echo "useage [start|stop|restart]"
        ;;
    esac
    
    # input_str 接收一个输入
    # 根据输入做对应操作 
    case "input_str" in
      Y | y)
        echo "yes"
        ;;
      N | n)
        echo "no"
        ;;
      *)
        echo "请输入： Y y N n"
        ;;
    esac
    
    # 除了上面两种分支用法外shell还有一种特殊的判断格式
    [ 条件 ] && 条件为真执行;条件为真执行  || 条件为假执行

#### shell 中判断条件的使用

在 if 这个用法中，条件的写法有多种

    如： 要表示 a 大于 b 且 a 小于 c
    在 (()) 和 [[]] 中 && 表示 与 ； || 表示 或
    在 [] 中 -a 表示 与 ；-o 表示 或
    并且要注意空格 每一项都要有空格
    
    if (( a > b )) && (( a < c ))
    
    if [[ $a > $b ]] && [[ $a < $c ]]
    
    if [ $a -gt $b -a $a -lt $c ]
    
    如： 要表示 a 大于 b 或 a 小于 c
    
    if (( a > b )) || (( a < c ))
    
    if [[ $a > $b ]] || [[ $a < $c ]]
    
    if [ $a -gt $b -o $a -lt $c ]














    

### 后记


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/wxgzh.jpg)







