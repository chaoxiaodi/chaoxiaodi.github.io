---
layout: post
title: "AWS boto3 python sdk auth practice"
subtitle: 'AWS boto3 认证最佳实践(个人认为)'
author: "chaoxiaodi"
header-style: text
tags:
  - AWS
  - boto3
  - auth
  - python
  - Credentials
---

# 前言

作为一个运维

并且公司使用AWS的云产品的时候

避免不了使用boto3来进行一些操作

如 同步机器 抓取监控数据等

本篇不写关于实际应用逻辑的内容

只关注boto3 在认证方面的一些逻辑

按照官方文档的说明 在使用认证的时候是按照一定的顺序进行查找的

    Boto3 will look in several locations when searching for credentials. The mechanism in which Boto3 looks for credentials is to search through a list of possible locations and stop as soon as it finds credentials. The order in which Boto3 searches for credentials is:

    1. Passing credentials as parameters in the boto.client() method
    2. Passing credentials as parameters when creating a Session object
    3. Environment variables
    4. Shared credential file (~/.aws/credentials)
    5. AWS config file (~/.aws/config)
    6. Assume Role provider
    7. Boto2 config file (/etc/boto.cfg and ~/.boto)
    8. Instance metadata service on an Amazon EC2 instance that has an IAM role configured.

按照我的理解把上面几条分成了3类

提供aksk类

提供配置文件类

绑定iam role类


# 代码

## 提供aksk

在最基础的使用场景中

使用 ACCESS_KEY SECRET_KEY 是最多 最常见的操作

官方也给出了对应的使用示例

    import boto3

    client = boto3.client(
        'ec2',
        aws_access_key_id=ACCESS_KEY,
        aws_secret_access_key=SECRET_KEY,
        aws_session_token=SESSION_TOKEN
    )

    import boto3

    session = boto3.Session(
        aws_access_key_id=ACCESS_KEY,
        aws_secret_access_key=SECRET_KEY,
        aws_session_token=SESSION_TOKEN
    )

这种使用方式最简单直接

但缺点也很明确 需要把key暴露在代码中

当然可以采取一些方法把key隐藏掉

比如把key 提前存到数据库 mysql redis中

比如把key 存到aws 托管的 secret manager中

使用环境变量的情况等与上面方式一致

## 提供配置文件

aws 同样提供了强大的命令行工具

要使用aws 命令行工具

必须在 ～/.aws/下配置相关内容

credentials 存放aksk

config 存放一些配置文件

当有多个账号/或者想要区分不同角色的时候可以通过config来实现

    credentials 内容示例

    [default]
    aws_access_key_id = AKxxxxxx
    aws_secret_access_key = xxxxxx

    config 内容示例

    多个账号

    [profile aws1]
    role_arn = arn:aws:iam::12345678901:role/manage-role
    region = us-west-2
    source_profile = default

    [profile aws2]
    role_arn = arn:aws:iam::12345678902:role/manage-role
    region = us-west-2
    source_profile = default


    多个角色

    [profile read-role]
    role_arn = arn:aws:iam::12345678901:role/read-role
    region = us-west-2
    source_profile = default

    [profile op-role]
    role_arn = arn:aws:iam::12345678901:role/op-role
    region = us-west-2
    source_profile = default

    可以通过给不同的role划分不同的权限

    在进行一些敏感操作的时候可以切换不同的role来避免一些操作失误问题

    aws的iam真心觉得非常强大

    可以通过指定 --profile 来切换对应的账号角色权限

    tips: 前提是完成iam sts相关权限的配置

有了上面的配置文件后

就可以使用profile_name来指定要切换role了

    import boto3

    session = boto3.Session(profile_name='aws2')
    dev_s3_client = session.client('ec2')

    同样直接使用了官方文档中给的代码示例

## 绑定instance-role

在创建ec2的时候可以通过高级设置里的 IAM instance profile来绑定一个IAM中配置的role

那么当前机器就自动拥有了role所配置的权限

无论是在aws 命令行工具使用

还是在boto3中使用 都不需要额外提供其他凭据

在官方文档中强烈推荐在ec2中使用instance-role来使用boto3

没有任何配置也就没有泄漏的问题

就算上述配置config的情况，如果机器允许多人登陆也是存在泄漏风险的

# 整合

上面介绍了常见的AWS boto3 认证的时候使用的方式

但是在日常开发过程中可能不同的场景需要用到不同的认证方式

比如本地开发的时候直接使用 aksk认证 或者 profile认证

等到发布到线上ec2的时候使用instance-role认证

那么就需要完成对认证方式一个简单整合

话不多说 直接上代码

哦！还有一点 aws 国内的服务与国外是不互通的需要单独适配下

    #!/usr/bin/env python

    import boto3

    class AwsApi(object):

        session = boto3.Session()

        def __init__(
            self,
            access_key: str = '',
            secret_key: str = '',
            profile_name: str = '',
            account_id: str = '',
            role: str = 'manage-role',
            is_global: bool = True):

            if access_key and secret_key:
                self.session = boto3.Session(
                    aws_access_key_id=access_key,
                    aws_secret_access_key=secret_key
                )

            if profile_name:
                self.session = boto3.Session(profile_name=profile_name)

            token = self.session.get_credentials().token
            if token is None and not account_id:
                raise Exception('local develop env need account to get token')

            if account_id:
                self.sync_account_role_token(account_id, role, is_global)

            if token:
                print('get token successful!')

        def sync_account_role_token(
            self,
            account_id: str,
            role: str,
            is_global: bool):

            if is_global:
                client = self.session.client('sts')
                arn = 'arn:aws:iam::%s:role/%s' % (account_id, role)
            else:
                client = self.session.client(
                    'sts',
                    endpoint_url='https://sts.cn-north-1.amazonaws.com.cn',
                    region_name='cn-north-1'
                )
                arn = 'arn:aws-cn:iam::' + str(account_id) + ':role/' + role
                arn = 'arn:aws-cn:iam::%s:role/%s' % (account_id, role)

            response = client.assume_role(
                RoleArn=arn,
                RoleSessionName='string',
                DurationSeconds=3600
            )

            access_key = response['Credentials']['AccessKeyId'] 
            access_secret = response['Credentials']['SecretAccessKey']
            access_token = response['Credentials']['SessionToken']
            token_expiration = response['Credentials']['Expiration']
            
            self.session = boto3.Session(
                aws_access_key_id=access_key,
                aws_secret_access_key=access_secret,
                aws_session_token=access_token
                )



    if __name__ == "__main__":
        # 应用示例1 不指定任何参数
        # 如果不指定参数的情况下默认按照文档中的顺序查找Credentials
        # 本地开发环境的时候一般是使用credential 文件中的 default的配置信息
        # 线上ec2的时候一般是使用instance-role
        aws_client = AwsApi()

        # 指定aksk
        # 那么最后应用的角色就是aksk所绑定的角色
        aws_client = AwsApi(access_key='akxxx', secret_key='skxxx')

        # 指定profile
        # 那么最后应用的角色跳转到aws2的角色
        aws_client = AwsApi(profile_name='aws2')

        # 如果同时使用了aksk profile 以profile为准

        # 跳转其他role
        # 将应用credential 文件中的 default的配置信息
        # 并跳转到账号 12312312312 的 manage-role 角色
        aws_client = AwsApi(account_id='12312312312')

这样一套简单的代码

后期使用的时候就能快速的完成aws的认证操作

不用每个场景都单独写一套逻辑

维护的时候还得同时维护多套

最关键的点是调用这段代码的时候不用更改逻辑

比如可以直接应用实例1调用 不管开发环境还是线上环境 都不需要对代码进行改动

只要提前维护好开发环境的Credentials

与线上环境的instance-role

# 参考
[AWS Boto3 Credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html)

Q：594934249

---我是超小弟·一名不务专业的秃头运维---

博客：[blog.chaoxiaodi.tech](https://blog.chaoxiaodi.tech)

github：[github:chaoxiaodi](https://github.com/chaoxiaodi)

微信公众号：老骥不伏枥只是近黄昏

![](/img/erweima.jpg)
