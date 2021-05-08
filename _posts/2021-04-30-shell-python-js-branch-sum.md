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

#### shell常用分支格式

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

#### shell判断条件的使用

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

#### python中条件分支语句使用

python 是通过缩进来控制代码块

所以常用的if语句格式如下

    # 单个分支
    if 条件:
        代码
        代码
   
    # 两个分支
    if 条件:
        代码
        代码
    else:
        代码
        代码

    # 多个分支
    # 可以有多个 elif
    if 条件:
        代码
        代码
    elif:
        代码
        代码   
    else:
        代码
        代码

#### python判断条件的使用

python 有许多的特有的语法特性

可以方便的进行分支语句的使用

在python使用变量进行判断时，变量必须进行定义否则会报错

    NameError: name 'var' is not defined

下面举一些常用的实例

##### 数字判断

    a = 0
    
    if a == 0:
        print('true')
    
    if a != 0:
        print('false')
        
    数字还支持 > < >= <= 
    

##### 字符串判断
##### 数组判断

对数组进行判断，一般情况下时通过 len 函数判断数组的长度是否大于 0 来判断数组是否存在值

但在python中可以直接判断

如果数组为空，则条件为假

    var = []
    if var:
        print('false')
    else：
        print('true,var')
        
    上面代码输出：true，var
        
    varr = [1]    
    if varr:
        print('true, varr')
    上面代码输出：true,varr
    
##### 判断是否是对应类型

python 提供了 `type` 函数来帮助进行变量类型的判断

可以根据 `type` 函数返回的结果判断是什么变量

除了 `type`函数外，还有 `isinstance` 函数可以帮助对变量类型进行判断

    # 使用 `type` 进行判断
    a = [1,2]
    if 'list' in str(type(a)):
        print('1')
    else:
        print('2')
        
    # 使用 `isinstance` 进行判断
    a = 1
    b = [1,2,3,4]
    c = (1,2,3,4)
    d = {‘a‘:1,‘b‘:2,‘c‘:3}
    e = "abc"
    if isinstance(a, int):
        print "a is int"
    else:
        print "a is not int"
    if isinstance(b, list):
        print "b is list"
    else:
        print "b is not list"
    if isinstance(c, tuple):
        print "c is tuple"
    else:
        print "c is not tuple"
    if isinstance(d, dict):
        print "d is dict"
    else:
        print "d is not dict"
    if isinstance(e, str):
        print "d is str"
    else:
        print "d is not str"

##### 逻辑判断 与 或

在 python 中使用 and or 关键字来进行逻辑的判断

    if a > b and a < c:
    
    if a > b or a < c:
    
除了使用上述关键字外

python 还提供了 not 关键字

可以使用 not 关键字进行条件结果的取反

    a = 2
    b = 3
    if a < b:
        print('true')
        
    if not a < b:
        print('false')
    else:
        print('true')
    
    # 可以结合 not 和 isinstance 进行变量类型判断
    if not isinstance(a, int):
        print('not int')
    else:
        print('int')
    代码输出：int
    









    

### 后记

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)

