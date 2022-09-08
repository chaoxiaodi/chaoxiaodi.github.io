---
layout: post
title: "记一次由于大意引发的bug"
subtitle: 'python 变量赋值多加逗号后变元组'
author: "chaoxiaodi"
header-style: text
tags:
  - python
---

### 前言

在更新一段代码功能时，由于疏忽大意产生了一个小小的bug

特此记录

### 经过

只截取部分代码块

原代码

    msg_body = []

    title = 'Notice: %s down %s' % (host_ip, at_person)
    msg_body.append(title)

    ---
    把上面的msg_body 作为参数传递给新的函数，新的函数中对msg_body进行处理
    str_msg = ''.join(msg_body)


新代码

    msg_body = []

    title = 'Notice: %s down' % host_ip,
    _at = '@%s' % at_person
    msg_body.append(title)
    msg_body.append(_at)

    ---
    把上面的msg_body 作为参数传递给新的函数，新的函数中对msg_body进行处理
    str_msg = ''.join(msg_body)

在代码运行过程中

会报错 TypeError: sequence item 0: expected str instance, tuple found

经过对msg_body进行打印输出

新代码的输出内容为

[(Notice: 1.1.1.1 down), '@133xxxxxxxx']

### 测试验证

经过测试

    a = 'a'
    b = 'b',
    c, d = 'c', 'd'
    e = 'e', 'e'
    f, g = 'f', 'g', 'x'

    a 的值为 字符串 a
    b 的值为 包含一个元素的 b 的元组
    c 的值为 字符串 c
    d 的值为 字符串 d
    e 的值为 包含两个元素 e 的元组
    当执行到 f g 行时会报错

### 后记

有时候在不经意见可能就犯了明知错误的错

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
