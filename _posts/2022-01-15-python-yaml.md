---
layout: post
title: "Python yaml 使用记录"
subtitle: 'python 与 yaml 使用过程中的记录'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - yaml
---

### 前言

最近的工作涉及到一个根据模版生成配置文件的需求

以前只是使用过yaml的配置文件格式，没有详细研究过yaml

本文使用python库来实现生产yaml格式的配置文件

#### 环境

    python 3.6+
    ruamel.yaml 0.17.19

关于yaml的库，python有pyyaml、ruamel.yaml等

[pyyaml_doc](https://pyyaml.org/wiki/PyYAMLDocumentation)

本文使用的是 ruamel.yaml

[ruamel.yaml_doc](https://yaml.readthedocs.io/en/latest/overview.html)

### 过程

根据模版生成配置文件

简而言之 就是读出yaml内容 进行加工后 再以yaml格式写入到文件

模版内容

    first:
      sencond_1:
        - '2-1'
        - '2-2'
      second_2:
        third:
        - '31'
        - '32'
      append_1:
      append_2:
      append_3:

#### 读

读 可以分为两步

1、open 读取文件

2、yaml库load内容

简单代码示例

    # 读取文件内容
    with open('test-template.yml', 'r', encoding='utf-8') as f:
        f_read = f.read()
        print(f_read)

    # 输出
    first:
      sencond_1:
        - '2-1'
        - '2-2'
      second_2:
        third:
        - '31'
        - '32'
      append_1:
      append_2:
      append_3:

    # 这一步输出内容会与原始文件保持一致

使用yaml库加载内容

    # yaml load 
    yaml_read = ruamel.yaml.safe_load(f_read)
    print(yaml_read)

    # 加载后的内容 差不多跟python中dict格式一般
    {'first': {'sencond_1': ['2-1', '2-2'], 'second_2': {'third': ['31', '32']}, 'append_1': None, 'append_2': None, 'append_3': None}}

    # 其实到这一步 就可以按照自己的需求对加载后的内容进行处理了

#### 写

模拟对加载的模版内容进行加工

    # 要追加的内容
    append = {
        "a": 'A',
        "b": 'B',
        "c": ['c', 'cc', 'ccc']
    }

    # 加工代码示例
    yaml_read['append_1'] = append
    yaml_read['append_2'] = append
    yaml_read['append_3'] = append
    print(yaml_read)

    # 输出加工后的内容
    {'first': {'sencond_1': ['2-1', '2-2'], 'second_2': {'third': ['31', '32']}, 'append_1': None, 'append_2': None, 'append_3': None}, 'append_1': {'a': 'A', 'b': 'B', 'c': ['c', 'cc', 'ccc']}, 'append_2': {'a': 'A', 'b': 'B', 'c': ['c', 'cc', 'ccc']}, 'append_3': {'a': 'A', 'b': 'B', 'c': ['c', 'cc', 'ccc']}}

对加工后的内容写入到文件

    # yaml 格式写入 类似于 json.dump
    yaml = ruamel.yaml.YAML()
    with open('output.yml', 'w', encoding='utf-8') as fo:
        yaml.dump(yaml_read, fo)

    # 写入后的文件格式
      first:
        sencond_1:
        - 2-1
        - 2-2
        second_2:
          third:
          - '31'
          - '32'
        append_1:
        append_2:
        append_3:
      append_1: &id001
        a: A
        b: B
        c:
        - c
        - cc
        - ccc
      append_2: *id001
      append_3: *id001

写入这一步输出的看起来怪怪的

看起来仿佛格式不对

而且有乱码生成

其实是完全符合yaml格式的

[yaml_doc](https://yaml.org/)

中间的id001 是ymal里的引用 


### 后记

    # 完整代码
    import ruamel.yaml

    append = {
        "a": 'A',
        "b": 'B',
        "c": ['c', 'cc', 'ccc']
    }

    def main():
        with open('test-template.yml', 'r', encoding='utf-8') as f:
            f_read = f.read()
            print(f_read)
        yaml_read = ruamel.yaml.safe_load(f_read)
        print(yaml_read)
        yaml_read['append_1'] = append
        yaml_read['append_2'] = append
        yaml_read['append_3'] = append
        print(yaml_read)
        yaml = ruamel.yaml.YAML()
        with open('output.yml', 'w', encoding='utf-8') as fo:
            yaml.dump(yaml_read, fo)


    if __name__ == "__main__":
        main()

如果有生成配置文件类

或者开发系统或脚本时

可以考虑使用yaml格式

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
