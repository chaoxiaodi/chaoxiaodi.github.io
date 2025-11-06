
---
layout: post
title: "Python字典优化技巧：使用setdefault简化代码"
subtitle: 'Python dict setdefault'
author: "chaoxiaodi"
header-style: text
tags:
  - Python
  - dict
  - code
---

### 前言
在Python开发中,我们经常需要处理字典(dict)数据结构。

有时会遇到需要先判断key是否存在,不存在则初始化再赋值的场景。

今天介绍一个优化技巧 - 使用dict.setdefault()方法来简化这类代码。

### 常见的判断初始化模式

我们先来看一个常见的代码模式:

    # 先判断key是否存在
    if key not in my_dict:
        # key不存在时初始化默认值
        my_dict[key] = []
    # 然后对值进行操作
    my_dict[key].append(new_value)

    # 先判断key是否存在
    if key not in my_dict:
        # key不存在时初始化默认值
        my_dict[key] = {}
    # 然后对值进行操作
    my_dict[key].update({"foo": "bar"})
    # 或者
    my_dict[key]["foo"] = "bar"

这种模式有以下几个特点:

- 需要先判断key是否存在
- 不存在时需要初始化默认值
- 最后对值进行操作
- 代码较为冗长

### setdefault方法简介

    setdefault(key, default=None, /)
    如果字典存在键 key ，返回它的值。
    如果不存在，插入值为 default 的键 key ，并返回 default 。 
    default 默认为 None。

### 代码优化示例

让我们看几个实际的优化示例:

#### 示例1: 初始化列表并追加

    # 优化前
    # 先判断key是否存在
    if key not in my_dict:
        # key不存在时初始化默认值
        my_dict[key] = []
    # 然后对值进行操作
    my_dict[key].append(new_value)

    # 优化后
    my_dict.setdefault(key, []).append(new_value)

#### 示例2: 初始化字典并设置属性

    # 优化前
    if key not in my_dict:
        my_dict[key] = {"aa": []}
    my_dict[key]['aa'].append(score)
    my_dict[key]['bb'] = ins['bb']

    # 优化后
    _ins_dict = instance_dict.setdefault(key, {"aa": []})
    _ins_dict['aa'].append(score)
    _ins_dict['bb'] = ins['state']

### 优势分析
使用setdefault()方法有以下优势:

代码更简洁,减少了条件判断

一行代码完成初始化和获取值

提高代码可读性

减少重复代码

#### 使用建议

虽然setdefault很好用,但也要注意以下几点:

default值会在每次调用时计算,如果是复杂对象要注意性能

对于简单的key检查和赋值,使用get()方法可能更合适

如果需要多次使用返回值,建议先赋值给变量再使用

### 总结
setdefault()是一个很实用的字典方法,可以帮助我们写出更简洁的代码。

但也要根据具体场景选择合适的使用方式,在代码简洁性和性能之间找到平衡点。

#### 建议在以下场景中使用setdefault:

需要初始化列表、字典等容器对象

需要连续操作字典中的值

代码中频繁出现key检查和初始化模式

通过合理使用setdefault,我们可以写出更pythonic的代码。

### 后记

#### 参考资料

- [python-dict-setdefault](https://docs.python.org/zh-cn/3.13/library/stdtypes.html#mapping-types-dict)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
