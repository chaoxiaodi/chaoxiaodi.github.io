---
layout: post
title: "google cloud python sdk 学习使用记录"
subtitle: '使用python调用google api'
author: "chaoxiaodi"
header-style: text
tags:
  - python
  - google
  - sdk
  - api
---

### 前言

工作中用到了Google Cloud Platform(GCP)

由于某些场景需要通过把某些GCP平台的功能集成到自己开发的平台中

如： 同步instance、刷新cdn等

在这之前没有使用过GCP的相关产品

经过一段时间学习测试

终于大致搞懂如何通过提供的sdk进行交互

可能是我才疏学浅

觉得这个GCP的文档写的真的是***

不说国内的阿里云和腾讯云

同样是国外的AWS在文档方面个人觉得写的比GCP好太多了

#### 简介

想要调用api

首先要安装、然后要处理权限认证、最后再根据不同服务调用不同方法

GCP不知道出于什么考虑

提供了两套api客户端库

[客户端库说明](https://cloud.google.com/apis/docs/client-libraries-explained)

Cloud 客户端库 [google-cloud-python](https://github.com/googleapis/google-cloud-python)

Google API客户端库 [google-api-python-client](https://github.com/googleapis/google-api-python-client)

按照官方说明

Cloud 客户端库比Google API客户端库更简单易用，且可以支持gRPC，用来提升性能

Cloud 客户端库进行了更细致的包划分

把服务分成了单独的pip包进行管理

经过测试，个人感受Cloud 客户端库确实更加简单易用


### 经过

两种客户端库的安装就不在单独写了

都是支持 pip install 

需要注意Cloud 客户端库要找到对应的库进行安装才行

下面对两个库的调用方法进行说明

#### 认证

在进行实际的使用客户端库之前先简单说明下google的认证

[使用客户端库向 Cloud 服务进行身份验证](https://cloud.google.com/docs/authentication/client-libraries)

按照官方文档说明

建议使用应用默认凭据 (ADC)提供凭据能大大简化代码量

官方提供了多种方案

一般开发采用把环境变量 `GOOGLE_APPLICATION_CREDENTIALS` 指向 client-key.json 文件

client-key.json文件一般来源于服务账号的key

关于服务账号以及角色权限等内容不在本文范围内

    # 如下，设置环境变量指向json文件
    os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = '/path/to/client-key.json'

#### Cloud 客户端库

以获取instance为例

    # 安装compute库
    pip install google-cloud-compute

    # 查看文档
    # [cloud compute docs]](https://cloud.google.com/compute/docs/reference/rest/v1/instances/list)
    # 根据文档 list方法需要project、zone两个必传参数
    # 本文只进行简单介绍，如有需求可根据文档处理其他参数
    # 由于使用的是环境变量提供的认证
    # 代码中不需要处理任何认证行为

    from google.cloud.compute_v1.services.instances import InstancesClient

    instance_client = InstancesClient()

    ins_list = instance_client.list(project=project, zone=region)
    print(ins_list)


#### Google API客户端库

同样以获取instance为例

    # 安装compute库
    pip install google-api-python-client

    # 查看文档 查看支持哪些api以及使用方法
    # [cloud compute docs]](https://github.com/googleapis/google-api-python-client/blob/main/docs/dyn/index.md)
    # [compute instances list方法使用](https://googleapis.github.io/google-api-python-client/docs/dyn/compute_v1.instances.html#list)
    # 根据文档 list方法需要project、zone两个必传参数, 最大返回数量参数示例
    # 本文只进行简单介绍，如有需求可根据文档处理其他参数
    # 由于使用的是环境变量提供的认证
    # 代码中不需要处理任何认证行为

    from googleapiclient.discovery import build

    service = build('compute', 'v1')
    instance_client = service.instances()
    instance_client.list(project=project, zone=region, maxResults=100)

    ins_list = instance_client.execute()
    print(ins_list)


#### 扩展：不使用ADC进行认证

基于安全性考虑

类似密钥，已经密钥文件之类的涉密信息

是不应该上传到代码仓库中的

下面介绍下不使用json进行认证的方法

json文件的内容是一个dict

可以直接把文件内容赋值给一个变量

然后使用google oauth2 进行认证

    from google.oauth2 import service_account

    account_info = {} # json 文件的内容

    # 生成一个credential 对象
    cred = service_account.Credentials.from_service_account_info(account_info)

    # Cloud 客户端库是在实例化Client对象时使用上面生成的credential对象
    instance_client = InstancesClient(credentials=cred)

    # Google API 客户端库是在实例化Client对象时使用上面生成的credential对象
    service = build('compute', 'v1', credentials=cred)


### 后记

本文涉及到的内容虽然不多

但是我将近用了一周的时间扣文档

才搞明白这个google api是怎么个用法

有了这个简单的例子

其他的使用方法几乎都能通用

学习记录


Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
